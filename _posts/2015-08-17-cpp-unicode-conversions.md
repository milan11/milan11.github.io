---
layout: post
title: C++ - Unicode conversions
---

How to convert strings in C++ between Unicode encodings (UTF-8, UTF-16 and UTF-32)? Here are some code snippets showing easiest ways I found out yet.

C++11 is required. You have to include:
{% highlight c++ %}
#include <codecvt>
#include <locale>
#include <string>
{% endhighlight %}

It correctly converts special cases such as:

- non BMP characters (2 code units in UTF-16)
- large code point values

## Firstly, a little helper function

{% highlight c++ %}
bool isLittleEndianSystem() {
	char16_t test = 0x0102;
	return (reinterpret_cast<char *>(&test))[0] == 0x02;
}
{% endhighlight %}

## Between UTF-8 and UTF-16

{% highlight c++ %}
std::u16string utf8_to_utf16(const std::string &s) {
	static bool littleEndian = isLittleEndianSystem();

	if (littleEndian) {
		std::wstring_convert<std::codecvt_utf8_utf16<char16_t, 0x10ffffU, std::codecvt_mode::little_endian>, char16_t> convert_le;
		return convert_le.from_bytes(s);
	} else {
		std::wstring_convert<std::codecvt_utf8_utf16<char16_t>, char16_t> convert_be;
		return convert_be.from_bytes(s);
	}
}
{% endhighlight %}

{% highlight c++ %}
std::string utf16_to_utf8(const std::u16string &s) {
	std::wstring_convert<std::codecvt_utf8_utf16<char16_t>, char16_t> convert;
	return convert.to_bytes(s);
}
{% endhighlight %}

## Between UTF-8 and UTF-32

{% highlight c++ %}
std::u32string utf8_to_utf32(const std::string &s) {
	std::wstring_convert<std::codecvt_utf8<char32_t>, char32_t> convert;
	return convert.from_bytes(s);
}
{% endhighlight %}

{% highlight c++ %}
std::string utf32_to_utf8(const std::u32string &s) {
	std::wstring_convert<std::codecvt_utf8<char32_t>, char32_t> convert;
	return convert.to_bytes(s);
}
{% endhighlight %}

## Between UTF-16 and UTF-32

There probably are better ways of doing this type of conversion than this (e.g. using UTF-8 as an intermediate encoding and using 2 of the previous code snippets).

{% highlight c++ %}
std::u32string utf16_to_utf32(const std::u16string &s) {
	std::string bytes;
	bytes.reserve(s.size() * 2);

	for (const char16_t c : s) {
		bytes.push_back(static_cast<char>(c / 256));
		bytes.push_back(static_cast<char>(c % 256));
	}

	std::wstring_convert<std::codecvt_utf16<char32_t>, char32_t> convert;
	return convert.from_bytes(bytes);
}
{% endhighlight %}

{% highlight c++ %}
std::u16string utf32_to_utf16(const std::u32string &s) {
	std::wstring_convert<std::codecvt_utf16<char32_t>, char32_t> convert;
	std::string bytes = convert.to_bytes(s);

	std::u16string result;
	result.reserve(bytes.size() / 2);

	for (size_t i = 0; i < bytes.size(); i += 2) {
		result.push_back(static_cast<char16_t>(static_cast<unsigned char>(bytes[i]) * 256 + static_cast<unsigned char>(bytes[i + 1])));
	}

	return result;
}
{% endhighlight %}

## Little and big endian

According to the [C++ standard](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4296.pdf), for ```codecvt_utf8``` and ```codecvt_utf8_utf16```, the endianess specification should not influence the behavior.

However, in ```utf8_to_utf16```, when compiling using ```-stdlib=libstdc++``` (libstdc++, version 5.2.0), I had to specify how the bytes will be stored in the underlying memory of ```char16_t``` type:

- ```std::codecvt_mode::little_endian``` - little endian (there will be e.g. 0x00 0x61 in memory for value 97)
- default (the third template argument of ```codecvt_x``` not specified) - big endian (there will be e.g. 0x61 0x00 in memory for value 97)

So, for little endian systems, we have to specify ```std::codecvt_mode::little_endian``` for the value of ```char16_t``` to be really 97 in that case.

When using ```-stdlib=libc++``` (libc++ version 3.6.2), the endianess specification was ignored and not influencing the output in any way (according to the standard).

So, the ```isLittleEndianSystem()``` distinction and the using of ```std::codecvt_mode::little_endian``` is here maybe temporarily for current ```-stdlib=libstdc++```.

Note that in ```utf16_to_utf32``` and ```utf32_to_utf16``` I was setting and getting the bytes explicitly (as big endian, which is the default), so not little/big endian distinction according to the system had to be made.

## Why not ```wchar_t``` and ```std::wstring```?

The types ```char``` (element type in ```std::string```), ```char16_t``` (element type in ```std::u16string```) and ```char32_t``` (element type in ```std::u32string```) have fixed sizes representing code unit size of the encoding we want to work with.

The ```wchar_t``` (element type in ```std::wstring```) type is platform specific, e.g.:

- 16 bit on Windows - used to encode UTF-16 (or, alternatively, UCS-2)
- 32 bit on Linux - used to encode UTF-32 (the same as UCS-4)

So, ```wchar_t``` does not have much value when we are discussing specific Unicode encodings. If you want to use it, see e.g. [codecvt_utf8](http://en.cppreference.com/w/cpp/locale/codecvt_utf8) (the table at the bottom - rows UTF-16, UCS2/UCS4).

## What about UCS-2?

Note that UCS-2 is not able to represent all Unicode code points. But it has an advantage of being a fixed length encoding (as opposite to UTF-16).

If you want to use it despite its disadvantage, here are the conversion functions:

{% highlight c++ %}
std::u16string utf8_to_ucs2(const std::string &s) {
	static bool littleEndian = isLittleEndianSystem();

	if (littleEndian) {
		std::wstring_convert<std::codecvt_utf8<char16_t, 0x10ffff, std::little_endian>, char16_t> convert_le;
		return convert_le.from_bytes(s);
	} else {
		std::wstring_convert<std::codecvt_utf8<char16_t>, char16_t> convert_be;
		return convert_be.from_bytes(s);
	}
}
{% endhighlight %}

{% highlight c++ %}
std::string ucs2_to_utf8(const std::u16string &s) {
	std::wstring_convert<std::codecvt_utf8<char16_t>, char16_t> convert;
	return convert.to_bytes(s);
}
{% endhighlight %}

{% highlight c++ %}
std::u16string utf16_to_ucs2(const std::u16string &s) {
	std::string bytes;
	bytes.reserve(s.size() * 2);

	for (const char16_t c : s) {
		bytes.push_back(static_cast<char>(c / 256));
		bytes.push_back(static_cast<char>(c % 256));
	}

	std::wstring_convert<std::codecvt_utf16<char16_t>, char16_t> convert;
	return convert.from_bytes(bytes);
}
{% endhighlight %}

{% highlight c++ %}
std::u16string ucs2_to_utf16(const std::u16string &s) {
	std::wstring_convert<std::codecvt_utf16<char16_t>, char16_t> convert;
	std::string bytes = convert.to_bytes(s);

	std::u16string result;
	result.reserve(bytes.size() / 2);

	for (size_t i = 0; i < bytes.size(); i += 2) {
		result.push_back(static_cast<char16_t>(static_cast<unsigned char>(bytes[i]) * 256 + static_cast<unsigned char>(bytes[i + 1])));
	}

	return result;
}
{% endhighlight %}

Be aware that if a code point being converted cannot be represented in UCS-2, the conversion can:

- produce invalid data (currently happened with ```-stdlib=libstdc++```)
- throw ```std::range_error``` (currently happened with ```-stdlib=libc++```)

## Testing

Tested on Linux:

- g++ (little endian system)
- clang++ -stdlib=libstdc++ (little endian system)
- clang++ -stdlib=libc++ (little endian and big endian system)

This code was used to test the conversion functions:

{% highlight c++ %}
struct TestString {
	TestString(std::string utf8, std::u16string utf16, std::u32string utf32)
		: utf8(std::move(utf8))
		, utf16(std::move(utf16))
		, utf32(std::move(utf32))
	{
	}

	std::string utf8;
	std::u16string utf16;
	std::u32string utf32;
};

void testOneString(const TestString &s) {
	if (utf8_to_utf16(s.utf8) != s.utf16)
		std::cout << "utf8_to_utf16" << std::endl;

	if (utf16_to_utf8(s.utf16) != s.utf8)
		std::cout << "utf16_to_utf8" << std::endl;

	if (utf8_to_utf32(s.utf8) != s.utf32)
		std::cout << "utf8_to_utf32" << std::endl;

	if (utf32_to_utf8(s.utf32) != s.utf8)
		std::cout << "utf32_to_utf8" << std::endl;

	if (utf16_to_utf32(s.utf16) != s.utf32)
		std::cout << "utf16_to_utf32" << std::endl;

	if (utf32_to_utf16(s.utf32) != s.utf16)
		std::cout << "utf32_to_utf16" << std::endl;

	try {
		if (utf8_to_ucs2(s.utf8) != s.utf16)
			std::cout << "utf8_to_ucs2" << std::endl;
	} catch (const std::range_error &) {
		std::cout << "utf8_to_ucs2 - range error" << std::endl;
	}

	try {
		if (ucs2_to_utf8(s.utf16) != s.utf8)
			std::cout << "ucs2_to_utf8" << std::endl;
	} catch (const std::range_error &) {
		std::cout << "ucs2_to_utf8 - range error" << std::endl;
	}

	try {
		if (utf16_to_ucs2(s.utf16) != s.utf16)
			std::cout << "utf16_to_ucs2" << std::endl;
	} catch (const std::range_error &) {
		std::cout << "utf16_to_ucs2 - range error" << std::endl;
	}

	try {
		if (ucs2_to_utf16(s.utf16) != s.utf16)
			std::cout << "ucs2_to_utf16" << std::endl;
	} catch (const std::range_error &) {
		std::cout << "ucs2_to_utf16 - range error" << std::endl;
	}
}

std::vector<TestString> strings
{
	TestString("\x61", u"\x0061", U"\x00000061"),
	TestString("\xEF\xBD\x81", u"\xFF41", U"\x0000FF41"),
	TestString("\xC4\x8D", u"\x010D", U"\x010D"),
	TestString("\x63\xCC\x8C", u"\x0063\x030C", U"\x00000063\x0000030C"),
	TestString("\xC4\xB3", u"\x0133", U"\x00000133"),
	TestString("\x69\x6A", u"\x0069\x006A", U"\x00000069\x0000006A"),
	TestString("\xCE\xA9", u"\x03A9", U"\x000003A9"),
	TestString("\xE2\x84\xA6", u"\x2126", U"\x00002126"),
	TestString("\xF0\x9D\x93\x83", u"\xD835\xDCC3", U"\x0001D4C3")
};

for (const TestString string : strings) {
	std::cout << string.utf8 << std::endl;
	testOneString(string);
}
{% endhighlight %}

There are some special instances tested, all described in [this previous post](http://milan11.github.io/unicode-examples/).

## See also

- [std::codecvt\_utf8\_utf16 - between UTF-8 and UTF-16](http://en.cppreference.com/w/cpp/locale/codecvt_utf8_utf16)
- [std::codecvt\_utf8 - between UTF-8 and UCS2/UCS4=UTF-32](http://en.cppreference.com/w/cpp/locale/codecvt_utf8)
- [std::codecvt\_utf16 - between UTF-16 and UCS2/UCS4=UCF-32](http://en.cppreference.com/w/cpp/locale/codecvt_utf16)
- [std::wstring\_convert](http://en.cppreference.com/w/cpp/locale/wstring_convert)

---
layout: post
title: Unicode examples
---

This post is mostly inspired by facts summarised at [utf8everywhere.org](http://utf8everywhere.org/).

Unicode is somehow complicated. There is a common misunderstanding about these things:

- there is not a simple direct mapping between graphemes (the user-perceived characters) and code points (the "Unicode"characters)
- UTF-16 is not a fixed width encoding - some code points are encoded into 2 and some into 4 bytes

There are some examples of this:

<table border="1"><tr><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;"></th><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;">Graphemes</th><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;">Code points</th><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;">UTF-8 bytes</th><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;">UTF-16 bytes</th><th style="border: 1px solid; white-space: nowrap; padding: 2px; font-weight: bold;">UTF-32 bytes</th></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">1. Simple</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x0061;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/61"><font color="0000FF">61</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>61</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 61</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 00 61</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">2. Fullwidth</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#xFF41;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/FF41"><font color="0000FF">FF41</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>EF</u> <u>BD</u> <u>81</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>FF 41</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 FF 41</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">3. Diacritic</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x010D;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/10D"><font color="0000FF">10D</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>C4</u> <u>8D</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>01 0D</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 01 0D</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">4. Diacritic - separate</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x0063;&#x030C;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/63"><font color="0000FF">63</font></a> <a href="http://unicode-table.com/en/30C"><font color="FF0000">30C</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>63</u></font> <font color="FF0000"><u>CC</u> <u>8C</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 63</u></font> <font color="FF0000"><u>03 0C</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 00 63</u></font> <font color="FF0000"><u>00 00 03 0C</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">5. Ligature</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x0133;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/133"><font color="0000FF">133</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>C4</u> <u>B3</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>01 33</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 01 33</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">6. Separate</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x0069;&#x006A;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/69"><font color="0000FF">69</font></a> <a href="http://unicode-table.com/en/6A"><font color="FF0000">6A</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>69</u></font> <font color="FF0000"><u>6A</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 69</u></font> <font color="FF0000"><u>00 6A</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 00 69</u></font> <font color="FF0000"><u>00 00 00 6A</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">7. Same grapheme</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x03A9;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/3A9"><font color="0000FF">3A9</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>CE</u> <u>A9</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>03 A9</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 03 A9</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">8. Same grapheme</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x2126;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/2126"><font color="0000FF">2126</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>E2</u> <u>84</u> <u>A6</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>21 26</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 00 21 26</u></font></td></tr><tr><td style="border: 1px solid; white-space: nowrap; padding: 2px;">9. Non-BMP</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;">&#x1D4C3;</td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><a href="http://unicode-table.com/en/1D4C3"><font color="0000FF">1D4C3</font></a></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>F0</u> <u>9D</u> <u>93</u> <u>83</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>D8 35</u> <u>DC C3</u></font></td><td style="border: 1px solid; white-space: nowrap; padding: 2px;"><font color="0000FF"><u>00 01 D4 C3</u></font></td></tr></table>

I looked up the information about the concrete characters using [unicode-table.com](http://unicode-table.com/). The code points in the table link to the information about the character on that site.

Each code unit is underlined separately. Each code point is encoded as:

- UTF-8 - 1-byte code units, from 1 to 6
- UTF-16 - 2-byte code units, 1 or 2
- UTF-32 - 4-byte code unit

UTF-16 and UTF-32 can be encoded as big or little endian. The table shows only the big endian variant (the most significant byte first).

All numbers are shown in hexadecimal form (even the code point numbers, where it maybe does not make any sense).

## 1. Simple

One grapheme as one code point encoded as 2 UTF-16 bytes. As we would expect.

## 2. Fullwidth

The same grapheme, but looks slightly different (is wider). This is used with some East Asian characters to [occupy the same width in fixed-width fonts](https://en.wikipedia.org/wiki/Halfwidth_and_fullwidth_forms).

## 3. Diacritic

The same, but for a less common and more complicated grapheme (with caron). We would expect this, too.

## 4. Diacritic - separate

Or we can use 2 separate code points to encode characters with diacritic:

- one for the base character (c) - blue color
- one for the diacritical mark (the caron) - red color

Note that this is the same grapheme (as users perceive it), but encoded in completely different ways. This can cause problems when comparing texts - the user would perceive them as equal, but the comparison of code points would tell that the texts are different. A process called normalization exists to convert the code points to the same form, so they can be correctly compared.

## 5. Ligature

These are 2 graphemes, which correspond to only one code point.

## 6. Separate

Of course, these graphemes (i and j) can be encoded separately, too - as two code points (blue and red).
The separate characters look almost the same as the ligature.

## 7./8. Same grapheme

These look exactly the same, but are different code points, with different meanings:

- Greek letter Omega
- Ohm sign

## 9. Non-BMP

This is the case where one code point is encoded as 4 UTF-16 bytes (instead of 2).

All characters which do not belong to the Basic Multilingual Plane (BMP) are encoded like this.

You can find other commonly used non-BMP characters in [this post at Stack Overflow](http://stackoverflow.com/a/5575000) - see the "trans-BMP code points" part.


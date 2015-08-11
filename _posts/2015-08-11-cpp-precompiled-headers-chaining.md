---
layout: post
title: C++ - precompiled headers chaining
---

Similarly to Microsoft Visual Studio compiler, gcc and clang compilers support precompiled headers, too. Let's look at creating and using of a precompiled header in clang compiler, particularly at chaining of precompiled headers (building a precompiled header using another precompiled header compiled earlier).

## Building a precompiled header

### Creating some included file

Firstly, we prepare the file [inc\_stl.h](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/inc_stl.h) which represents something:

- we would normally include in the code using the ```#include``` directive
- we want to use in a precompiled header (because it is e.g. included more times or the compilation of it takes long)

This file includes a lot of stuff (standard C++ libraries, including STL) so that it takes longer to compile (so we can compare compilation times more conveniently). Of course, normally you would include only one item directly instead of such package, e.g. ```<iostream>```, and use such one item in a precompiled header.

### Creating prefix header file

The prefix header is a header file that is compiled into the precompiled header. It contains inclusion of all the needed header files (the files we want to optimize the compilation for) - in our case only the inc\_stl.h. We name this prefix header [pch\_stl.h](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/pch_stl.h).

### Compiling

The prefix header can be compiled to the precompiled header using [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/create_pch_stl.sh):

{% highlight bash %}
clang++ -std=c++11 -x c++-header pch_stl.h -o out/pch_stl.h.pch
{% endhighlight %}

So, we compile pch\_stl.h (which includes inc\_stl.h) into pch\_stl.h.pch and it takes 1.3 seconds:

![Building a precompiled header]({{ site.baseurl }}/images/cpp-precompiled-headers-chaining/create_pch_stl.svg)

## Using the precompiled header

### Creating a program file

The program [project\_using\_stl](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/project_using_stl.cpp) wants to use inc\_stl.h without the need to recompile it every time the program source code is being compiled.

So, the precompiled header (which has the inc\_stl.h already compiled) can be used - we build the project using [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/build_project_using_stl.sh):

{% highlight bash %}
clang++ -std=c++11 -include out/pch_stl.h project_using_stl.cpp -o out/project_using_stl
{% endhighlight %}

The precompiled header is included in the build process and the compilation of the project takes only 0.3 seconds:

![Using the precompiled header]({{ site.baseurl }}/images/cpp-precompiled-headers-chaining/build_project_using_stl.svg)

For comparison, building of the project without the precompiled header took 1.3 seconds - [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/build_project_using_stl__without_pch.sh) has been used:

{% highlight bash %}
clang++ -std=c++11 project_using_stl.cpp -o out/project_using_stl
{% endhighlight %}

## Building a chained precompiled header using another precompiled header

What if we want to create a precompiled header which contains something more in addition to the headers we already built into a precompiled header?

We will compile the prefix header [pch\_stl\_and\_boost.h](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/pch_stl_and_boost.h) which contains:

- inc\_stl.h which we already have built as a precompiled header pch\_stl.h.pch
- [inc\_boost.h](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/inc_boost.h) which represents some new additional things we want to compile into a precompiled header (some things from boost are there which take long to compile)

Similarly to building of the program file, we can use a precompiled header while building another precompiled header - using [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/create_pch_stl_and_boost.sh):

{% highlight bash %}
clang++ -std=c++11 -include out/pch_stl.h -x c++-header pch_stl_and_boost.h -o out/pch_stl_and_boost.h.pch
{% endhighlight %}

It produces pch\_stl\_and\_boost.h.pch and takes 7.0 seconds:

![Building a precompiled header using another precompiled header]({{ site.baseurl }}/images/cpp-precompiled-headers-chaining/create_pch_stl_and_boost.svg)

However, when not using the precompiled header (so compiling the C++ libraries in addition to boost), it takes 8.3 seconds - [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/create_pch_stl_and_boost__without_pch.sh) has been used:

{% highlight bash %}
clang++ -std=c++11 -x c++-header pch_stl_and_boost.h -o out/pch_stl_and_boost.h.pch
{% endhighlight %}

## Using the chained precompiled header

We can use such precompiled header (including both C++ libraries and boost) to build [this project](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/project_using_stl_and_boost.cpp).

Use [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/build_project_using_stl_and_boost.sh):

{% highlight bash %}
clang++ -std=c++11 -include out/pch_stl_and_boost.h project_using_stl_and_boost.cpp -o out/project_using_stl_and_boost
{% endhighlight %}

![Using the chained precompiled header]({{ site.baseurl }}/images/cpp-precompiled-headers-chaining/build_project_using_stl_and_boost.svg)

It takes only 0.7 seconds. For comparison:

- [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/build_project_using_stl_and_boost__without_pch.sh) builds it without precompiled header - it takes 6.5 seconds
- [this command](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining/blob/master/build_project_using_stl_and_boost__partial_pch.sh) builds it with the first precompiled header (containing C++ libraries, without boost) - it takes 5.7 seconds.

## Conclusion

The most important thing is that you can use a precompiled header while building another precompiled header. This allows creating of some hierarchy of code from things that rarely change to things that change more often and rebuilding only the code (and precompiled headers) that have been changed while using the alredy built precompiled headers that did not change.

Maybe you wonder why precompiled header compilation took 1.3 and compiling project using the precompiled header took 0.3, while compiling project without precompiled header took 1.3 (so not 1.6 as the sum of precompiled header and project alone). This is because the precompiled header has to contain all the information from the header files - no code can be left out. Contrary to this, the inclusion of the header in the program file can be optimized, since most of the code included in the headers is often not used. Only the code that is needed has to be compiled into the resulting file and maybe the compiler delays some operations while including the header until really needed.

All files and actions described in this post are available in [the repository experiments-cpp-precompiled-headers-chaining](https://github.com/milan11/experiments-cpp-precompiled-headers-chaining). The header files produce a message while compiling so you can observe what is being built and what not.


---
layout: post
title: C++ - throw vs. throw e
---

When rethrowing an exception in C++, there can really be a difference between ```throw``` and ```throw e;```.

## Prepare some classes

{% highlight c++ %}
struct B {
	B() {};
	B(const B & other) { std::cout << "copy to B" << std::endl; }
	B(B && other) { std::cout << "move to B" << std::endl; }
	B & operator==(const B & other) = delete;
	B & operator==(B && other) = delete;

	virtual ~B() {};
	virtual void print() const { std::cout << "B" << std::endl; }
};

struct D : B {
	D() {};
	D(const D & other) { std::cout << "copy to D" << std::endl; }
	D(D && other) { std::cout << "move to D" << std::endl; }
	D & operator==(const D & other) = delete;
	D & operator==(D && other) = delete;

	virtual void print() const { std::cout << "D" << std::endl; }
};
{% endhighlight %}

- these are ```struct```s so that everything is public - it does not matter here
- this is a simple Base-Derived example with a virtual member function printing some output so that we know which one was called
- constructors (copy and move) print output, so that we know which one is called (or if none was called)
- assignment operators (copy and move) are deleted so that we avoid any confusion

## Simple test

{% highlight c++ %}
void f(const B & e) {
	e.print();
}
{% endhighlight %}

{% highlight c++ %}
	f(D());
{% endhighlight %}

We call a function which has a parameter of type ```B``` and we use the type ```D``` as an argument. There is nothing interesting here, we do it only to test if the virtual function works correctly. It prints:

```
D
```

Of course, no copy/move constructor had been called, because we were passing by reference.

### Rethrowing using throw

{% highlight c++ %}
	try {
		try {
			throw D();
		} catch (const D & e) {
			throw;
		}
	} catch (const B & e) {
		e.print();
	}
{% endhighlight %}

Here, the code takes these steps:

- throw ```D```
- catch it as ```D```
- rethrow it
- catch it as ```B```
- call its ```print```

Since it is ```D```, it prints:

```
D
```

This was expected. It is similar to calling of the ```f``` function above.

### "Rethrowing" using throw e

{% highlight c++ %}
	try {
		try {
			throw D();
		} catch (const D & e) {
			throw e;
		}
	} catch (const B & e) {
		e.print();
	}
{% endhighlight %}

The only difference is that ```throw;``` has been changed to ```throw e;```. It prints:

```
copy to D
D
```

It's still ```D```, but an (unnecessary) copy has been made (via copy constructor). In fact, ```throw``` always needs a copy constructor to be accessible. Even the previous example (with ```throw``` only) would not work with the copy constructor deleted, because the line ```throw D();``` would need it - despite not really calling it due to copy elision.

### "Rethrowing" D using throw e

{% highlight c++ %}
	try {
		try {
			throw D();
		} catch (const B & e) {
			throw e;
		}
	} catch (const B & e) {
		e.print();
	}
{% endhighlight %}

Now, additionally, in the first catch, we are catching it as ```B``` instead of ```D```. This prints:

```
copy to B
B
```

Not only there is a copy, but the type had been changed entirely to the base type (we caught). So, the throwing took into account the type of the variable, not the real type of the object.

Please note that:

- if we used ```throw;``` instead of ```throw e;``` here, it would behave as expected - no type change and even no copy
- if we caught ```D``` instead of ```B``` in the second (outer) catch, it would miss this catch, because the new thrown type really is ```B```

### Conclusion

About ```throw;```:

- rethrows an exception
- does not make a copy
- preserves type
- additionally, can be used from ```catch (...)```

About ```throw e;```:

- throws a new exception (to be correct, this should not be called *rethrowing*)
- does make a copy
- does not preserve type

So, a simple hint is to always use ```throw;```.

Please note that the statement about not making a copy is only true when catching a reference. Catching a value of course does cause a copy of the argument value into the catch block.

### See also

This is explained in several posts on Stack Overflow, e.g.

- [C++: Throwing an exception invokes the copy constructor?](http://stackoverflow.com/questions/10855506/c-throwing-an-exception-invokes-the-copy-constructor)
- [copy constructor and throw-expression](http://stackoverflow.com/questions/12356332/copy-constructor-and-throw-expression)

Especially, [this answer](http://stackoverflow.com/a/10855545) summarises the main points made here.

The difference between ```throw;``` and ```throw e;``` is explained [here at cppreference.com](http://en.cppreference.com/w/cpp/language/throw) - see the two cases in the Syntax part and their descriptions in the Explanation part.


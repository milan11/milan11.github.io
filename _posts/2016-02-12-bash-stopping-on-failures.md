---
layout: post
title: Bash - stopping on failures
---

A good habit is to begin each bash script with:

- ```set -e``` (stop execution if a command returns with non-zero exit status)
- ```set -u``` (stop execution if an undefined variable has been used, instead of treating the variable to have an empty value)

However, the ```set -e``` commnd has some consequences. The script can end where we would not expect it:

{% highlight bash %}

# A sub-command returns non-zero exit status

man non_existing_entry # error (as expected)
OUTPUT=$(man non_existing_entry) # error (maybe not expected)
man non_existing_entry & # NOT error

# Result of an arithmetic expression is zero

let NUM=0 # error (not expected)
NUM=1
let NUM=NUM-1 # error (not expected)

NUM=0 # NOT error
NUM=$((0)) # NOT error
NUM=1
NUM=$((NUM-1)) # NOT error

# grep did not find anything

ip addr | grep non_existing # error (not expected)

{% endhighlight %}

We can avoid treating these cases as errors by surrounding the commands by ```set +e``` and ```set -e```:
{% highlight bash %}
set +e
man non_existing_entry
let NUM=0
ip addr | grep non_existing
set -e
{% endhighlight %}

A function doing this arround a command can help to simplify the code:
{% highlight bash %}
noerror()
{
	set +e
	$@
	set -e
}
{% endhighlight %}

It can be used as follows:

{% highlight bash %}
noerror man non_existing_entry
ip addr | noerror grep non_existing
{% endhighlight %}

---
layout: post
title: Android - how to open menu on old Android emulator
---

Older Android versions used a separate button to open a menu (which was then replaced by the action bar and its overflow menu). When testing applications using an emulator with some old Android, we often want to somehow access the menu. The emulators used to provide a button for this, but such buttons were shown only in a specific skin: Skin with dynamic hardware controls.

I created an emulator with a very old Android (API level 5) and now, with current updated Android SDK, somehow such buttons are not shown and even the keyboard shortcuts (F2 or Page up) do not work.

I tried to open the menu using adb:
{% highlight bash %}
adb shell input keyvevent KEYCODE_MENU
{% endhighlight %}

It ended up with a very strange response:

```
[1]   Killed                  input keyevent K...
```

The solution is simple - use a numeric key code:
{% highlight bash %}
adb shell input keyevent 82
{% endhighlight %}

If somehow that won't work either, here is another solution:
{% highlight bash %}
adb shell chmod 666 /dev/input/event0
adb shell sendevent /dev/input/event0 1 139 1
adb shell sendevent /dev/input/event0 1 139 0
{% endhighlight %}

You can find another emulator keyboard shortcuts e.g. in [this post at Stack Overflow](http://stackoverflow.com/a/14606911).

The solution using /dev/input/event0 is based on [this Stack Overflow post](http://superuser.com/a/962048). This can be used for simulating multiple buttons at once or long pressing of a button, e.g. backspace for 2 seconds:
{% highlight bash %}
adb shell sendevent /dev/input/event0 1 14 1
sleep 2
adb shell sendevent /dev/input/event0 1 14 0
{% endhighlight %}


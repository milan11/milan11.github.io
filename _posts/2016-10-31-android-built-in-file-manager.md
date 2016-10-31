---
layout: post
title: Android - built-in file manager
---

In Android 7 (maybe older versions, too), at least on Nexus phones, there is a file manager with some basic functionality.

However, it is quite inconvenient to access it - we have to go deep into Settings -> Storage -> Explore.

It is basically the Downloads application, as can be seen when opening the recent tasks list. But opening Downloads using its launcher does not provide such functionality anywhere.

This file manager has some basic functions:

- searching by file name (recursively, not only in current directory)
- 2 view modes - list and cards
- sorting by name / date modified / size
- selecting multiple files at once (including Select all)
- deleting
- copying/moving
- renaming
- opening files

Copying and moving leads us to select the destination directory. However, it can happen that the internal storage is not accessible here - you will probably have only 2 options: Recent and Drive. There is a simple trick: go to Recent and in the overflow menu select Show internal storage.

To have the file manager better accessible, create a simple application which will serve as a launcher for it. Create an application wich one empty activity and add the following code to the onCreate method of that activity:
{% highlight java %}
Intent intent = new Intent("android.provider.action.BROWSE");
intent.setComponent(new ComponentName("com.android.documentsui", "com.android.documentsui.FilesActivity"));
intent.setData(Uri.parse("content://com.android.externalstorage.documents/root/primary"));
startActivity(intent);
finish();
{% endhighlight %}

I obtained the needed parameters of the Intent from adb logcat. It prints the following after choosing the Explore item in Settings -> Storage:

```
ActivityManager: START u0 {act=android.provider.action.BROWSE cat=[android.intent.category.DEFAULT] dat=content://com.android.externalstorage.documents/root/primary cmp=com.android.documentsui/.FilesActivity (has extras)}
```


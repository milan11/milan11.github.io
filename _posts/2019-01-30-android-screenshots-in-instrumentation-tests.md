---
layout: post
title: Android - screenshots in instrumentation tests
---

After taking a screenshot in instrumentation tests, we often need it to be transferred back to the host computer. Here I explain different approaches I found.

To capture a screenshot during instrumentation test, we can use this function:

{% highlight kotlin %}
fun screenshot(name: String) {
    val capture = Screenshot.capture()
    capture.name = name
    capture.format = CompressFormat.PNG

    val processors = HashSet<ScreenCaptureProcessor>()
    processors.add(BasicScreenCaptureProcessor())

    capture.process(processors)
}
{% endhighlight %}

However, we easily end up with an error:
```java.io.FileNotFoundException: /storage/emulated/0/Pictures/screenshots/aaa-3167d90a-1344-4e7f-952d-286b72098005.png (Permission denied)```

Here I summarize options how to deal with this.

## Grant the storage write permission

Use this rule in the test class:
{% highlight kotlin %}
@Rule
@JvmField
var permissionRule = GrantPermissionRule.grant(Manifest.permission.WRITE_EXTERNAL_STORAGE)
{% endhighlight %}

The files can be downloaded using ```adb pull```.

**Problems with this approach:**

In instrumentation tests, there are 2 APKs involved:

- the real app (the app being tested) APK
- the test app APK

The permission has to be declared in the manifest of the app being tested:
{% highlight xml %}
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
{% endhighlight %}
For me, it did not work when added to the manifest in the androidTest source set (to appear only in the test app APK). It really had to be declared in the manifest of the real app. But you can use one of there workarounds:

- Declare the permission only in the manifest for debug configuration of the real app (or in another configuration or product flavor intended for tests). I did not want to use this workaround because I wanted to test exactly the same configuration which will be released (without any, even subtle, modifications).
- Declare the permission in the manifest of the test app (androidTest source set) and use ```sharedUserId``` for the two APKs to share permissions. This is described in more detail [here](https://stackoverflow.com/a/14196493/7164302).

## Save screenshots to app data directory

Of course, we do not need any special permission for this.

To change the directory where the screenshots will be saved, we have to change the value of the protected member ```mDefaultScreenshotPath``` of ```BasicScreenCaptureProcessor```:
{% highlight kotlin %}
class AppDataScreenCaptureProcessor : BasicScreenCaptureProcessor {
    constructor(context : Context) {
        mDefaultScreenshotPath = context.dataDir
    }
}
{% endhighlight %}

Then, use this new processor:
{% highlight kotlin %}
processors.add(AppDataScreenCaptureProcessor(context!!))
{% endhighlight %}

**Problems with this approach:**

How to download the files?
For a debug build, [we can use](https://stackoverflow.com/a/36841916/7164302) e.g.:
{% highlight bash %}
adb shell "run-as example.app.package.id cat /data/user/0/example.app.package.id/aaa-f682ea1d-95ce-44f3-8f1e-2431a4a1cbed.png" > /tmp/screenshot.png
{% endhighlight %}

For a release build, [we can use](https://stackoverflow.com/a/15559278/7164302) e.g.:
{% highlight bash %}
adb backup  -f /tmp/screenshots example.app.package.id
{% endhighlight %}
But this prints
```Now unlock your device and confirm the backup operation...```
and waits for the operation to be approved on the phone. I did not do any later experiments here about how to automate this, but maybe it is possible to automate it somehow. 

![Confirming backup]({{ site.baseurl }}/images/android-screenshots-in-instrumentation-tests/backup.png)

## Use adb to take the screenshot

{% highlight bash %}
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png /tmp/screenshot.png
{% endhighlight %}

**Problems with this approach:**

We want the screenshots to be initiated by the instrumentation tests code - the screenshots have to be taken at the right time. Only the instrumentation test code knows what is currently displayed and what needs to be captured. This could be done by some signaling - the instrumentation tests can send some signal to the host computer (e.g. by writing something special to the logs visible using ```adb logcat```), and as soon as the host computer sees the signal, it will create the screenshot.

## Upload screenshots to HTTP server

This is the approach I ended up using.

From the emulator, the host computer is accessible at IP address ```10.0.2.2``` and there is a high probability our app actually uses the INTERNET permission, so there are no additional permissions to be added only because of the tests.

Our special capture processor uploads the screenshots, instead of writing them to the filesystem:

{% highlight kotlin %}
class HttpUploadScreenCaptureProcessor : BasicScreenCaptureProcessor() {

    override fun process(capture: ScreenCapture): String? {
        var filename = if (capture.name == null) defaultFilename else getFilename(capture.name)
        filename += "." + capture.format.toString().toLowerCase()

        val port = if (BuildConfig.DEBUG) 10004 else 10005
        val app = BuildConfig.APPLICATION_ID

        val url = URL("http://10.0.2.2:$port/$app/$filename")
        val httpCon = url.openConnection() as HttpURLConnection
        httpCon.requestMethod = "PUT"
        httpCon.doOutput = true

        httpCon.outputStream.use {
            capture.bitmap.compress(capture.format, 100, it)
        }

        run {
            val code = httpCon.responseCode
            if (code != 200) throw Exception("Invalid response: $code")
        }

        return filename
    }

}
{% endhighlight %}

One more thing we need to do is to run some HTTP server supporting PUT on our host computer.
Here is some example using Python 2:

{% highlight python %}
#!/usr/bin/python2
import SimpleHTTPServer
import SocketServer
import os
import sys

class SputHTTPRequestHandler(SimpleHTTPServer.SimpleHTTPRequestHandler):
    def do_PUT(self):
        print self.headers
        length = int(self.headers["Content-Length"])
        path = self.translate_path(self.path)
        parent = os.path.dirname(path)
        try:
            os.makedirs(parent)
        except OSError:
            if not os.path.isdir(parent):
                raise
        with open(path, "wb") as dst:
            dst.write(self.rfile.read(length))
        self.send_response(200, "")
        self.end_headers()

PORT = int(sys.argv[1])

Handler = SputHTTPRequestHandler

httpd = SocketServer.TCPServer(("127.0.0.1", PORT), Handler)

print "serving at port", PORT
httpd.serve_forever()
{% endhighlight %}

This code was inspired by [A simple HTTP Server supporting put](https://gist.github.com/fabiand/5628006).

This is a very simplified solution:

- the HTTP server is accessible by anyone without authentication
- the HTTP server still supports e.g. GET, which automatically returns files from our filesystem (this functionality is not needed in our scenario)
- there are probably even more security problems

So, make sure to use it only for testing purposes, and at least make sure to run it listening only on the loopback interface (```127.0.0.1```).

Run the HTTP server before running tests in a directory where you want the screenshots to be put:
{% highlight bash %}
http_server.py 10004
{% endhighlight %}
(or ```10005``` for the Release configuration)

This approach works well with Jenkins. Use ```Execcute shell``` build step:
{% highlight bash %}

# run HTTP server (will end when this script ends)
mkdir screenshots
(cd screenshots && /var/lib/jenkins/scripts/http_server.py 10005) &

# run tests
ANDROID_SERIAL=emulator-5580 android/gradlew -p android -PTEST_BUILD_TYPE=release connectedAndroidTest
{% endhighlight %}

Adding **Archive the artifacts** post-build action will archive the collected screenshots. Just specify ```screenshots/``` as **Files to archive**.
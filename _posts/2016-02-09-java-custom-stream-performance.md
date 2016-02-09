---
layout: post
title: Java - custom stream performance
---

To implement a custom input stream in Java, the only mandatory method to implement is ```read()```. However, a stream with only ```read()``` implemented can perform very badly.

I created an imput stream whose sole purpose was to constrain the original stream length (to provide only a part of the original bytes).

{% highlight java %}
public class PartInputStream extends InputStream {

        private InputStream orig;
        private int pos;
        private int length;

        public PartInputStream(InputStream orig, int length) {
                this.orig = orig;
                this.length = length;
        }

        @Override
        public int read() throws IOException {
                if (pos == length) {
                        return -1;
                } else {
                        ++pos;
                        return orig.read();
                }
        }
}
{% endhighlight %}

It implemented only ```read()```, leaving ```read(byte[], int, int)``` inherited from InputStream. According to the [Java InputStream documentation](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html#read-byte:A-int-int-), the default implementation of ```read(byte[], int, int)``` only calls read() repeatedly to prepare the requested data chunk byte by byte.

In my case, the custom input stream was set atop of InflaterInputStream, CipherInputStream and finally FileInputStream where the data came from.

The callers often called ```read(byte[], int, int)``` to get chunks of data. However, it lead to calling ```read()``` repeatedly and querying the lower streams one byte at a time. Many unnecessary calls had to be made in my stream. Even worse, the lower streams had to respond to many calls separately instead of returning bigger chunk of data (which they had probably prepared anyway).

The correct implementation with noticeably better performance is:

{% highlight java %}
public class PartInputStream extends InputStream {

        private InputStream orig;
        private int pos;
        private int length;

        public PartInputStream(InputStream orig, int length) {
                this.orig = orig;
                this.length = length;
        }

        @Override
        public int read() throws IOException {
                if (pos == length) {
                        return -1;
                } else {
                        ++pos;
                        return orig.read();
                }
        }

        @Override
        public int read(byte[] buffer, int byteOffset, int byteCount) throws IOException {
                if (byteCount == 0) {
                        return 0;
                }

                int bytesLeft = length - pos;
                if (bytesLeft == 0) {
                        return -1;
                }

                int toRead = (bytesLeft < byteCount) ? bytesLeft : byteCount;

                int reallyRead = orig.read(buffer, byteOffset, toRead);
                if (reallyRead == -1) {
                        throw new IOException("Inconsistent stream lengths");
                }

                pos += reallyRead;

                return reallyRead;
        }
}
{% endhighlight %}

There is a similar case with the ```skip(long)``` method of InputStream.

Additionally, default ```read(byte)``` calls ```read(byte, int, int)```, so by implementing ```read(byte)``` we can avoid this one additional call.

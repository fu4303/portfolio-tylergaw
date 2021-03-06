---
tags: post
layout: "layouts/article.njk"
highlightSyntax: true
title: "WebVTT Cuemarker"
date: "2013-06-08"
meta:
  description:
    The Web Video Text Track format is easy enough to work with, but marking
    cue times can be a bit of a chore. I wrote a small tool to make it a
    little easier.
---

        <p class="entry-intro">
            <a href="http://dev.w3.org/html5/webvtt/">WebVTT</a> is a text format
            that can be used to provide captions and subtitles for HTML video.
            While working on a project that uses it I found myself needing a
            way to find and mark precise cue in/out times.
            <a href="https://github.com/tylergaw/webvtt-cuemarker">webvtt-cuemarker</a>
            is a small tool I whipped up in JavaScript to do just that.
        </p>
        <h2>A WebVTT Primer</h2>
        <p>
            The format is still fairly new and not yet widely supported, so I'll give
            a quick example of how it's used.
        </p>
        <pre><code class="language-clike">WEBVTT FILE

00:00:01.265 --> 00:00:06.210
This is text that will appear over the video at during the in/out points.

00:00:05.500 --> 00:00:10.250
You can use <b>HTML</b> within WebVTT, pretty cool.
</code></pre>

<p>
That's an example of a very simple WebVTT file with two cues. We'll
refer to it as <code>captions.vtt</code>
</p>
<p>
The next thing to do is to let your HTML video know about the WebVTT file.
That's done using the <code>track</code> element.
</p>
<pre><code class="language-markup">&lt;video controls&gt;
&lt;source src="/path/to/video.mp4" type="video/mp4"&gt;
&lt;source src="/path/to/video.webm" type="video/webm"&gt;
&lt;track kind="captions" label="English captions"
src="/path/to/captions.vtt" srclang="en" default>&lt;/track&gt;
&lt;/video&gt;
</code></pre>
<p>
That's all there is to it. If we played the fictional video above
we would see the two captions appear at the times provided in <code>captions.vtt</code>.
For a more in-depth look at the <code>track</code> element I suggest
<a href="http://www.html5rocks.com/en/tutorials/track/basics/">Getting Started With the Track Element</a>
on HTML5 Rocks. That's where I first learned about all this goodness.
</p>
<h2>Why Is Cuemarker Needed?</h2>
<p>
WebVTT is very specific about the time format for cues. Cue times must
be represented exactly as shown in <code>captions.vtt</code>. It's
easy to type out the times in that way, but how do you find
precise times using HTML video? HTML video controls display the time
of the video, but only down to the second. And not in the format needed
for WebVTT. This is why I wrote Cuemarker.
</p>
<p>
Cuemarker provides a keyboard shortcut???the period key???for setting in/out
points and outputting the cue time in the required format.
</p>
<p>
Cuemarker also provides shortcuts to interact with HTML
video. You can play/pause using the space bar and
seek forward/backward using the right/left arrow keys. Those controls
allow you to arrive at more precise moments in video than you can
using the default video scrubber provided by the browser.
</p>
<h2>How Do I Use It?</h2>
<p>
I'll go back to the HTML in the previous example, but I'm going to
remove the <code>track</code> element. For marking cue times you only
need an HTML video.
</p>
<pre><code class="language-markup">&lt;video controls&gt;
&lt;source src="/path/to/video.mp4" type="video/mp4"&gt;
&lt;source src="/path/to/video.webm" type="video/webm"&gt;
&lt;/video&gt;
</code></pre>
<p>
Once your HTML is in place, you'll need a bit of JavaScript.
Cuemarker creates a variable???<code>cuemarker</code>???in the global scope.
<code>cuemarker</code> is a function that takes one required parameter;
the video element. It also takes a second, optional parameter to
specify a seek interval and output function.
</p>
<pre><code class="language-javascript">cuemarker(document.querySelector('video'));
</code></pre>
<p>
That's a bare-bones example and may be all that you'll need. The cue
times will be output using <code>console.log</code>.
</p>
<p>
Here's another example using both the available options.
</p>
<pre><code class="language-javascript">cuemarker(document.querySelector('video'), {
seekInterval: 0.5 //default is 0.03,
output: function (cuetime) {
var times = document.getElementById('cuetimes'),
li = document.createElement('li');
li.innerHTML = cuetime;
times.appendChild(li);
}
});
</code></pre>
<p>
In that example, the seek interval is less precise than the default
and the cue times will be injected into the page as <code>li</code>
elements that are children of the <code>ul</code> with an id of "cuetimes".
</p>
<figure>
<img src='https://tylergaw.com/articles/assets/post-image-cuemarker-output.jpg' alt='Screenshot of Cuemarker output'>
<figcaption>Cuemarker output</figcaption>
</figure>
<p>
Cuemarker doesn't try to get cute by attempting to transfer cue times to
a WebVTT file. All it's interested in doing is outputting the
in/out times. Once you've marked in/out times you
can copy them over to your WebVTT files. That's how I've been using
it and it's working really well for me.
</p>
<p>
A working demo of the latter example is located
<a href="http://tylergaw.github.io/webvtt-cuemarker/demo.html">here</a>.
</p>
<h2>That's It?</h2>
<p>
Yep, that's what I got for now. Again, the <code>track</code> element
and WebVTT are not yet widely supported. As of this writing only the
latest Chrome and Opera are cool enough to do so. The Web moves fast though and
I know it won't be long before Firefox and Safari follow suit.
</p>
<p>
Limited support for the underlying technology means that this is a
tool you may not need today, but you might tomorrow.
<a href="https://github.com/tylergaw/webvtt-cuemarker">Fork it!</a>
</p>
<p>
<i>Thanks for reading</i>
</p>

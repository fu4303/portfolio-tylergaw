---
tags: post
layout: "layouts/article.njk"
highlightSyntax: true
title: "Reacting to Media Queries in JavaScript"
date: "2012-05-17"
meta:
  description: I was looking for a way for Javascript to know when a media query
    condition was met or not met. While window.matchMedia provides the main
    functionality, I wanted something that was a little more automatic.
---

<p>
        <b>Brass tacks:</b> <em><a href="http://tylergaw.github.com/media-query-events/">A demo</a> and <a href="https://github.com/tylergaw/media-query-events">some code</a>.</em>
</p>
<p class="entry-intro">
        <code>window.matchMedia</code> provides a way for Javascript to react when a media query condition is met or unmet. While the functionality it allows is great, the necessary code duplication required to use it leaves a bit to be desired. I'm going to walk through a work-in-progress approach to getting around that duplication.
</p>
<h2>window.matchMedia</h2>
<p>
        The method is simple enough to use and works the way you'd expect it. You give it a media query string it gives you back a <a href="https://developer.mozilla.org/en/DOM/MediaQueryList">MediaQueryList</a> object.
</p>
<pre><code class="language-javascript">var mql = window.matchMedia("(min-width: 480px)");</code></pre>
<p>
        That sets the value of <code>mql</code> to a MediaQueryList object with two members, something like:
</p>
<pre><code class="language-javascript">MediaQueryList: {
  matches: true,
  media: "(min-width: 480px)"
}</code></pre>

<p>
The boolean value of the <code>matches</code> member will be determined by the width of your browser window at the time.
</p>
<p>
You can add event listeners to MediaQueryList objects. An event will fire each time the condition is triggered. This allows you to be updated on the status of the media query without having to resort to polling or a <code>window.resize</code> event. Using <code>mql</code> from above we can set up a listener and handler like so:
</p>
<pre><code class="language-javascript">mql.addListener(handleMediaChange);
handleMediaChange(mql);

var handleMediaChange = function (mediaQueryList) {
if (mediaQueryList.matches) {
// The browser window is at least 480px wide
}
else {
// The browser window is less than 480px wide
}
}</code></pre>

<p class="note">
You can find similar code examples and further explanation of MatchMedia on the <a href="https://developer.mozilla.org/en/DOM/window.matchMedia">Mozilla Developer Network</a>
</p>
<p>
MatchMedia works pretty well. As usual there are some caveats that I'll mention later. My issue is with needing to specify the media query when you create the MediaQueryList. If you wanted an event to fire for every media query you might have, you'd have manually copy each from your stylesheets to you JS. Every time you update a media query in CSS, you'd have to do the same in JS. What I want is a script to look at a page's stylesheets, pick out all the media queries and create a MediaQueryList for each one.
</p>
<h2>
Getting Media Queries from CSS to JS
</h2>
<p>
When I first started looking into this I thought I was in for some really hairy stuff. I imagined myself having to make ajax requests to fetch stylesheets, use weird regexes to find the <code>@media</code> rules, and employ other types of not-so-fun things. Luckily, this did not turn out to be the case.
</p>
<p>
I've written small script that accomplishes the tasks I'm after, <a href="https://github.com/tylergaw/media-query-events/blob/master/js/mq-events.js">mqEvents.js is on Github</a> and a working example is at <a href="http://tylergaw.github.com/media-query-events/">tylergaw.github.com/media-query-events</a>.
</p>
<pre><code class="language-javascript">(function () {
var mqEvents = function (mediaChangeHandler) {
var sheets = document.styleSheets,
numSheets = sheets.length,
mqls = {},
mediaChange = function (mql) {
console.log(mql);
}

        if (mediaChangeHandler) {
            mediaChange = mediaChangeHandler;
        }

        for (var i = 0; i < numSheets; i += 1) {
            var rules = sheets[i].cssRules,
                    numRules = rules.length;

            for (var j = 0; j < numRules; j += 1) {
                if (rules[j].constructor === CSSMediaRule) {
                    mqls['mql' + j] = window.matchMedia(rules[j].media.mediaText);
                    mqls['mql' + j].addListener(mediaChange);
                    mediaChange(mqls['mql' + j]);
                }
            }
        }
    }

    window.mqEvents = mqEvents;

}());</code></pre>

<p>
I'm going to go through the code here and explain what's happening each step of the way.
</p>
<pre><code class="language-javascript">var mqEvents = function (mediaChangeHandler)</code></pre>
<p>
The <code>mqEvents</code> function takes a single parameter, a function that will be called each time a media query is triggered.
</p>
<pre><code class="language-javascript">var sheets = document.styleSheets,
numSheets = sheets.length</code></pre>
<p>
The document contains an object of all loaded stylesheets. Our <code>sheets</code> variable is a list of <a href="https://developer.mozilla.org/en/DOM/stylesheet">StyleSheet</a> objects. <code>numSheets</code> is stored for convenience for when we loop over the list of stylesheets.
</p>
<pre><code class="language-javascript">mediaChange = function (mql) {
console.log(mql);
}

if (mediaChangeHandler) {
mediaChange = mediaChangeHandler;
}
</code></pre>

<p>
If the <code>mediaChangeHandler</code> argument is not passed to <code>mqEvents</code>, a default function, <code>mediaChange</code> will handle each media query event. The default doesn't do much of anything. For the purpose of this script we just want to have something there.
</p>
<pre><code class="language-javascript">for (var i = 0; i < numSheets; i += 1) {
var rules = sheets[i].cssRules,
numRules = rules.length;</code></pre>
<p>
Here we're looping over our <code>sheets</code> list to look at each loaded stylesheet. The <code>rules</code> variable is list of all the rules of the current stylesheet represented as <a href="https://developer.mozilla.org/en/DOM/cssRule">CSSRule</a> objects. <br><em>This is where things take a turn for the awesome.</em>
</p>
<p>
At the start of this I was aware that all a document's stylesheets could be accessed and that all the rules of the stylesheets were represented by CSSRule objects, but what I didn't know was that CSSRule objects that contain a media query have a unique name. Take a look at this screen shot of the console when logging out each CSSRule object:
</p>
<figure>
<img src='https://tylergaw.com/articles/assets/mqevents-cssrules-in-console.jpg' alt='Screenshot of the Chrome developer console showing a number of CSSRule objects being logged.'>
<figcaption>Logging out each CSSRule object reveals a unique name for media query rules. Badass.</figcaption>
</figure>
<p>
Regular CSS rules have the name "CSSStyleRule" while media queries have the name "CSSMediaRule". This is great because it gives us an easy way to pluck out only the media queries from our stylesheets without needing to resort to string parsing, which can get ugly quickly.
</p>
<p class="note">
(It looks like other types of rules like font-face and keyframes have <a href="https://developer.mozilla.org/en/DOM/cssRule#section_2">unique names too</a>.)
</p>
<pre><code class="language-javascript">for (var j = 0; j < numRules; j += 1) {
if (rules[j].constructor === CSSMediaRule)</code></pre>
<p>
We can now loop over each rule in the stylesheet and check to see if it is a media query. The condition here, checking to see if the constructor matches the name "CSSMediaRule", was also new to me. I found that approach in a thorough <a href="http://stackoverflow.com/a/332429/368634">Stack Overflow answer</a> on the topic.
</p>
<pre><code class="language-javascript">mqls['mql' + j] = window.matchMedia(rules[j].media.mediaText);
mqls['mql' + j].addListener(mediaChange);
mediaChange(mqls['mql' + j]);</code></pre>
<p>
Now that we know we're only dealing with media queries, we're free to use them with matchMedia to create MediaQueryList objects, bind events to those and handle them with a given handler function. This bit of code is nearly the same as the matchMedia example. The noticeable exception here is that each MediaQueryList object is being added to a hash, <code>mqls</code>, that we created earlier. We could accomplish the same thing without putting each object in the hash, this was more of a forward-thinking thing. I have a feeling that there could be a use for holding on to all of the objects to access them later.
</p>
<h2>Usage</h2>
<p>
So why would you use this and what happens when you do? In the demo I linked to, this is the implementation:
</p>
<pre><code class="language-javascript">var msg = document.getElementById('condition'),
handleMediaChange = function (mql) {

    // For some reason Firefox has trouble always running this code.
    // The console.log seems to help it.
    // TODO: Figure out what the hell that's all about
    console.log();

    if (mql.matches) {
        msg.setAttribute('class', 'met');
        msg.innerHTML = 'The condition "' + mql.media + '" was met.';
    }
    else {
        msg.setAttribute('class', 'unmet');
        msg.innerHTML = 'The condition "' + mql.media + '" was not met.';
    }

};

mqEvents(handleMediaChange);</code></pre>

<p>
Notice the big comment about Firefox there, I still don't know what that is. Like I said, work-in-progress here.
</p>
<p>
What this does is update a the DOM element, represented by <code>msg</code>, each time a media query is triggered. <code>mqEvents</code> doesn't try to react differently to specific media queries. It calls the same handler each time one is triggered. The handler function, however, does receive information about the media query. It knows what the media query is???which I'm placing in the DOM with <code>mql.media</code>???and if the condition was met by the window???which I'm checking with <code>mql.matches</code>.
</p>
<h2>What Else Can This Be Used For?</h2>
<p>
Maybe parts of the page layout are done with Javascript and need to be updated each time a media query is triggered. Maybe it could be used to do conditional loading of images, scripts, fonts, or the like. This type of conditional loading of assets could be used to help reduce the bandwidth usage on mobile devices.
</p>
<h2>Concerns and Caveats</h2>
<p>
If a page has a lot of media queries, <code>mqEvents</code> is going to add a lot of event listeners and it's going to be calling the handling function a lot of times. I haven't run into anything problematic with my small demo page, but I'd be curious to see the impacts on performance on a page with a substantial number of media queries.
</p>
<p>
With all these fancy new things browser support is a big question. <code>matchMedia</code> is not supported very well yet. According to <a href="http://caniuse.com/#search=matchMedia">caniuse.com</a> we're looking at Chrome 17+, Firefox 9+, Safari 5.1+, iOS Safari 5.0+, IE 10+ (with an ms prefix), and no support yet for Opera. For unsupported browsers a polyfill could be employed. Here's a good one right <a href="https://github.com/paulirish/matchMedia.js/">here</a>.
</p>
<h2>Other Cool Stuff</h2>
<p>
As I was working on this I found out about a related project named <a href="http://harvesthq.github.com/harvey">Harvey</a>. I haven't used it yet, but it looks really good. It looks like it still has that duplication of media queries issue, but if you're looking for targeted reactions to specific media queries it might be the way to go.
</p>
<p>
<i>Thanks for reading</i>
</p>

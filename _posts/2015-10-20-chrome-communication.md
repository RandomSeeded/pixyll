---
layout:     post
title:      Chrome Extensions and iFrames
date:       2015-10-20
summary:    Your guide to things that shouldn't be allowed, but totally are
categories: Chrome Javascript iFrames
---

A chrome extension I'm currently working on has some somewhat...unusual...goals. We want to be able to invisibly load a page in the background of the user's browser, and then interact with it as that user. Specifically, we want to be able to use the Facebook Messenger, but for very good reasons (spam), Facebook has removed the ability to read and send messages through the Facebook API for quite some time.

So how can we get around these (sensible) restrictions inside a Chrome extension? By doing things that shouldn't be allowed, but totally are.

## Thing that is terrible #1: loading pages invisibly

Chrome extensions have the ability to load pages invisibly, in the background. They are actually loaded the moment the user logs on to his computer, and persist after all Chrome windows are closed, unless the Chrome process is killed. Opening a link to any website the extension wants with injected javascript is as easy as placing an iFrame. 

{% highlight html %}
<script src=bower_components/jquery/dist/jquery.js></script>
<script src=background.js></script>
<iframe src="https://facebook.com/messages/" id="iframe"/>
{% endhighlight %}

But wait! Facebook (or any other site of your choice) doesn't allow itself to be loaded in an iFrame?

## Thing that is terrible #2: stripping headers

This is the browser, we can do whatever we want. Facebook doesn't get a say:

{% highlight js %}
chrome.webRequest.onHeadersReceived.addListener(
    function(info) {
        var headers = info.responseHeaders;
        for (var i=headers.length-1; i>=0; --i) {
            var header = headers[i].name.toLowerCase();
            if (header == 'x-frame-options' || header == 'frame-options') {
                headers.splice(i, 1); // Remove header
            }
        }
        return {responseHeaders: headers};
    },
    {
        urls: [ '*://*/*' ], // Pattern to match all http(s) pages
        types: [ 'sub_frame' ]
    },
    ['blocking', 'responseHeaders']
);
{% endhighlight %}

Bam. No more iFrame deny headers here! (You should probably set this to match only the specific site that you want to strip the headers from).

OK, great, we've got an iFrame loaded. How can we get the content out of that iFrame? How can we read those messages?

## Thing that is terrible #3: content script communication

The extension has the capability to inject any javascript we specify into any frame we like. While it doesn't have the capability to interact with the script files that already exist for that frame, it **does** have the capability of interacting with the DOM. Injecting javascript:

manifest.json:
{% highlight json %}
"content_scripts": [
  {
    "matches": [
        "*://www.facebook.com/*"
        ],
    "js": ["bower_components/jquery/dist/jquery.js","facebook.js"],
    "run_at": "document_end",
    "all_frames": true
  },
{% endhighlight %}


Once that javascript is injected, it also has the ability to communicate with the outside world:

facebook.js:
{% highlight js %}
chrome.runtime.sendMessage(/* Send anything you want here, including page contents*/, 
  function(response) {
    // And do anything you want here
  }
);
{% endhighlight %}

...which is listened to in the background.js:
{% highlight js %}
chrome.runtime.onMessage.addListener(function(message) {
  // Do anything you want with that data here.
});
{% endhighlight %}

## What have we achieved?

We're able to load an page, invisibly, in the background. This page can be interacted with via injected javascript. This injected javascript has the ability to speak with external javascript files.

## What's the point?

For the extension I'm working on, this allows the sending and receipt of messages without a Facebook API. A hidden facebook page is opened in the background and interacted with, allowing the creation of an alternate chat client which will interact with the facebook service.

## Doesn't this all sound scary?

You bet. I had no idea before starting this project the scope of what Chrome extensions were permitted to do. To be clear, this is in no way specific to Facebook; if the user grants permissions, a Chrome extension can load, read, and interact with ANY page, invisibly, as you, using your existing session information.

This could include your social media, your bank, or anything else. Be creative! Be afraid. Maybe, just maybe, actually read the permissions you give to an extension the next time you go to install one, because if you're not scared of the power of malicious extensions after reading this, you should be.

---
layout:     post
title:      Socket.io with Path Names
date:       2015-12-05 13:38:00
summary:    Hosting multiple socket.io servers on the same server
categories: Javascript Socket.IO Apache Pathname
---

##The Problem

Imagine you have the following snippet of code:

{% highlight js %}
var socket = io.connect('http://www.example.com:1337/pathname');
{% endhighlight %}

You might expect that it would attempt to connect to a socket server located at `http://www.example.com:1337/pathname`...but you'd be wrong. Instead, any requests to the server are actually still sent to `http://www.example.com:1337/socket.io`. Now, generally speaking, this isn't a problem. You send to `/socket.io`, the server listens for `/socket.io`. However, this gets to be a PITA the moment you try to have multiple applications, all hosted on the same server, which all are listening for incoming socket connections.

Imagine you're NateWillard.com, premier source of all things me. There are multiple socket.io-using projects hosted here, including the project-in-24-hours multiplayer game [Knockout](/projects/warlocks/) and the interactive jukebox Glitch (still to be uploaded). Requests for both, by default, will be routed to `www.natewillard.com/socket.io`. How can we tell where to route these requests?  Path prefixes! Here's how:

##The Code

*server.js:*
{% highlight js %}
var io = require('socket.io')(server, { path: '/projects/warlocks/socket.io' });
{% endhighlight %}

*client.js:*
{% highlight js %}
var socket = io.connect(window.location.origin, { path: 'projects/warlocks/' };
{% endhighlight %}

And, finally, if we're using an additional web server, we'll need to make sure that it's capable of correctly passing on our requests. Here I'm using Apache:

*000-sites-enabled.conf:*
{% highlight apache %}
ProxyPreserveHost On
ProxyPass /projects/warlocks/socket.io/ http://127.0.0.1:1337/projects/warlocks/socket.io/
ProxyPassReverse /projects/warlocks/socket.io/ http://127.0.0.1:1337/projects/warlocks/socket.io/
ProxyPass /projects/warlocks/ http://127.0.0.1:1337/
ProxyPassReverse /projects/warlocks/ http://127.0.0.1:1337/
{% endhighlight %}

The above is going to redirect any requests to projects/warlocks/ to our node static file server, sans prefix. However, any socket requests are going to be piped through with the prefix remaining, to be used as an specific identifier for the project requested.

Enjoy!


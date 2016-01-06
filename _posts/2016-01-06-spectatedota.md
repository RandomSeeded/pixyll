---
layout:     post
title:      "Introducing: Spectate Dota"
date:       2016-01-06 20:28:30
summary:    Live-Streaming Dota, 24/7
categories: Dota2 Steam Javascript
---

##[Check it out!](http://twitch.tv/spectatedota)

##What is it?

It's a 3-part bot:

**1)** It spoofs the Steam and Dota2 clients in order to interact with the Steam servers and Dota 2 game coordinator. It does this via the extraordinarily helpful NPM module [node-dota2](https://github.com/RJacksonm1/node-dota2), which is an extension of [node-steam](https://github.com/seishun/node-steam), which is a port of the truly wonderful [SteamKit2](https://github.com/SteamRE/SteamKit). Deep thanks go out to all the developers who have made all of these libraries both possible and available.

**2)** It uses an actual Dota2 game client and [Open Broadcaster Software](https://obsproject.com/), in combination with spoofed input via xdotool, to live-stream whatever match is selected via the spoofed game client from part #1.

**3)** It uses a node IRC bot (also integrated with part #1) in order to make information about the currently streaming game available to any spectators. Go ahead and try these commands out: they're !mmr and !players. 

I'll release the code publicly once I've had an opportunity to clean it up (and remove any personally identifying information). In the meantime, enjoy!

##Suggestions?

[Let me know!](/blog/contact)

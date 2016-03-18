---
layout:     post
title:      Introducing TwitchBotDetector v2
date:       2016-03-17 20:39:00
summary:    A modification/improvement for TwitchBotDetector
categories: twitch viewbots promises
---

Today I'm proud to introduce [my own version of TwitchBotDetector](https://github.com/RandomSeeded/TwitchViewbotCatcher)

## The result

![Viewbots Detected](/blog/images/viewbotDetected.png)

## The background

I came across [this very cool repo](https://github.com/popcorncolonel/TwitchBotDetector) the other day. It attempts to identify channels on Twitch which are using viewbots to artificually increase their ranking on Twitch. These bots are run on the principle that if you increase your fake viewers, you'll appear higher on the Twitch page, and therefore get more real viewers. With enough real viewers, you get a partnership, and all that $.

Notable (denied) examples of this include Athene, Winter, and Massan.

## How it works:

Most viewbots, or at least the cheap ones, don't chat. If I were to guess as to why, my intuition would be that the viewbots are using headless browsers which don't have the capability. I could very well be wrong. Some viewbot providers do have chatting capability, but with a limited range of responses. If your botes can only broadcast the same messages, Kappa spam aside, this is probably an even clearer sign to the average viewer that the viewer counts are inflated.

One way to detect viewbots, therefore, is to compare the number of 'viewers' to the number of people in chat. 

## Why re-code it?

I was originally going to just make modifications to the original TwitchBotDetector. However, the baked-in synchronous issuance of hundreds of API requests was going to be difficult to remove. It was far easier instead to simply re-code in Node. The requests can be run synchronously, and simply re-requested in case of API throttling. 

## What's next?

1. An IRC module which will engage in the actual public shaming service announcements. This will go into the IRC channel of the viewbotting streamer in question and make a notice informing the chat of the botting.
  * False-positive mitigation. Things important to look for include:
  * Hosting of the stream on 3rd party sites
2. Streams that have just started/closed down can have misleading counts (double-check after period of time)
Other? Let me know!
3. Modification such that this can be run as permanent process, sans cron-job
4. Right now the code is using promises in a very cool but pretty insane way. While this was great practice, it's pretty unreadable. Code cleanup is coming.

## Thoughts?

[Let me know!](/blog/contact)


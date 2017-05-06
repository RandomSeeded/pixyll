---
layout:     post
title:      "Introducing Surf Alert"
date:       2017-03-18 00:00:00
summary:    Erlang OTP Simple One For One Examples
categories: Erlang OTP Surfpings
---

General Idea here:

Use dumb inits

Because if you put logic in the init, and the init breaks, it may break over and over and over again, and you will be sad.

In surf pings, what happens if you put the API call in the init? It tries, it breaks. Every time it breaks, its runner breaks (linked). This is intentional.

Its parent should be restarted, but shouldnt try again until the next day. What went wrong?

In surf pings, what WOULD have happened if you put the API call in a gen_server:call? It would fail, and that would kill the parent. The parent would be restarted, but would not retry until the next day.

Actually this is all completely irrelevant.

What actually happened is: I had a dumb supervisor setting, and that killed everything, and then I was sad.


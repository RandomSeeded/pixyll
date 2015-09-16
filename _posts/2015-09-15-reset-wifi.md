---
layout:     post
title:      Reset wifi, the tiny bash script
date:       2015-09-09
summary:    Because I need to do it 12 times a day
categories: node-steam Steam Steam-guard javascript node
---

## The rationale 

Because the HR wifi periodically stops giving me internet.

## The code

{% highlight bash %}
#!/bin/bash
# Reset network manager (for use when connection is poor)
nmcli n off
sleep 1
nmcli n on
{% endhighlight %}

## Star it on github

[You know you want to](https://github.com/RandomSeeded/Reset-Wifi)


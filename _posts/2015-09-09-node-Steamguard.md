---
layout:     post
title:      Authenticating through Steam Guard with Node-Steam
date:       2015-09-09
summary:    A walkthrough
categories: node-steam Steam Steam-guard javascript node
---

***Note:** if you're reading this well after post-date, there is a high likelihood that the information in this walkthrough is no longer accurate. While you can still use it as a general guide, odds are you will have to do some reading of the [repo source](https://github.com/seishun/node-steam) and [protobufs wiki](https://github.com/seishun/node-steam/wiki/Protobufs) in order to figure out what changes you may need to make.*

## Authenticating with Steam Guard: Overview

The general process for logging into Steam for an account with Steam Guard enabled is the following:

1. Attempt to login to your account with your username and password on an unrecognized machine
2. Steam will request a secondary authorization code which they have sent to your email, to verify that you are who you say you are.
3. Re-login, this time additionally with the secondary authorization code provided
4. Steam will accept your login, and additionally reply with a unique identifier for this machine
5. All future logins can use this unique identifier, and thereby avoid having to re-generate and re-provide the secondary authorization codes for every login.

## Authentication with Steam Guard: the Code

After installing node-steam (from the [github](https://github.com/seishun/node-steam) or via `npm install steam`), you can run the following as an example of how to connect. This is a working, Steam Guard-compatible modification of steam-guard's provided example.js.

{% highlight js %}
var fs = require('fs'); 
var Steam = require('steam');
var crypto = require('crypto');

var steamClient = new Steam.SteamClient();
var steamUser = new Steam.SteamUser(steamClient);
var steamFriends = new Steam.SteamFriends(steamClient);

steamClient.connect();
steamUser.on('updateMachineAuth', function(sentry, callback) {
  console.log('writing to file sentry');
  fs.writeFileSync('sentry', sentry.bytes);
  callback({ sha_file: getSHA1(sentry.bytes) });
});
function getSHA1(bytes) {
  var shasum = crypto.createHash('sha1');
  shasum.end(bytes);
  return shasum.read();
}
steamClient.on('connected', function() {
  steamUser.logOn({
    account_name: '[redacted]',
    password: '[redacted]',
    // un-comment this when providing secondary auth-code from email
    // auth_code: '[redacted]', 
    sha_sentryfile: getSHA1(fs.readFileSync('sentry'))
  });
});

steamClient.on('logOnResponse', function(logonResp) {
  console.log(logonResp);
  if (logonResp.eresult == Steam.EResult.OK) {
    console.log('Logged in!');
    // Your code here
  }
});
{% endhighlight %}

## What's going on here / How to use this

We are attempting to connect to Steam by running steamClient.connect(). The first time we attempt to do this, we'll receive back error 63, which corresponds to secondary authorization code required. Check your email. Place that authorization code in the `auth_code` field, and re-connect. On the next attempt to connect, the `updateMachineAuth` event will be triggered, Steam will send a unique machine identifier to us, and we will write that to a file in our directory named 'sentry.' We now remove the `auth_code` field from our logOn code. All future logons will read the sentry file, generate a SHA1 hash representing that file, and send that hash to Steam as a unique machine identifier. We're in! Have fun :)


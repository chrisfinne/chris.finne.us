---
layout: post
title: Reminder To Setup Iptables and hosts After Cloning Slicehost
date: 2010-04-13
---

I've been bitten by this a few times. I've been running Ubuntu slices recently on Slicehost. I generally follow the [Pickled Onion setup Article](http://articles.slicehost.com/2008/4/25/ubuntu-hardy-setup-page-1)

But when I clone a slice, the clone gets some fresh files:
    /etc/network/interfaces
    /etc/hosts

The "interfaces" file is the hook used to load up the iptables on boot, so that is particularly important.

I asked Slicehost support about this and they said that cloning basically sets up a new image. I though that was a pretty lame answer. If all my libraries in /usr and /opt are in there, I don't consider it a "new image".


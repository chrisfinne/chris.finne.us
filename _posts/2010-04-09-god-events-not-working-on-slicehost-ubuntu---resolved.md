---
layout: post
title: God Events not working on Slicehost Ubuntu - Resolved
date: 2010/04/09
---

So I'm installing [God](http://god.rubyforge.org/) (which for me is now the #1 search result (SERPS) when I'm logged into Google, but #9 when I'm logged out.)

I want to use the event-based monitoring, i.e. transitions in God-speak, but they aren't working.

On my **Ubuntu 8.10 (intrepid)** slice that I imaged last year, it runs the kernel:

**2.6.24-23-xen #1 SMP Mon Jan 26 03:09:12 UTC 2009 x86_64 GNU/Linux**

<pre>
root@jobs:~ # god check
using event system: netlink
starting event handler
forking off new process
forked process with pid = 7531
killing process
[fail] never received process exit event
</pre>

And the god.log says:
<pre>
* Event conditions are not available for your installation of god.
</pre>

Whereas on a recently installed Ubuntu 9.10 (karmic) Slice with the kernel:

**2.6.32.9-rscloud #6 SMP Thu Mar 11 14:32:05 UTC 2010 x86_64 GNU/Linux**

<pre>
root@aerialink:~# god check
using event system: netlink
starting event handler
forking off new process
forked process with pid = 2893
killing process
[ok] process exit event received
</pre>

So I've upgraded the kernel through Slicehost's web UI (which does an immediate reboot, FYI). The reboot brought God back up (hey, looks like the init.d script works) which kicked off the DelayedJobs, so looks like everything is groovy now.

Could [Slicehost](http://www.slicehost.com/) make kernel upgrades any easier? I feel like George Jetson. I actually had to click a button!

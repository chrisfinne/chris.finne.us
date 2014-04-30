---
layout: post
title: Best Web Site Uptime Monitoring Service
date: 2010/05/03
---

# site24x7
I've been using [site24x7](http://www.site24x7.com/) for years and they have been a reliable vendor for monitoring the uptime of my websites. They have been the cheapest vendor I've seen, although there are countless ones out there. I'm not counting the free services as they usually don't have a frequent enough polling cycle for my tastes. My stable, unchanging sites, I typically poll every 30-60 minutes. The ones I actively work on I'll poll every 2-15 minutes.

Recently, my site went down just minutes after I went to sleep. I had recently kicked off a job that created a lot of temp files and working at the startup-pace, I got sloppy and forgot to check the free space. I hadn't setup a disk space monitor yet either, but that was a moot point as I was away from email at the time anyhow. The server also had only 1 disk partition as it was just a virtual slice and I hadn't take the time to partition log files separately (again startup priorities at play here). The result was 5 hours of down time at a peak period. **Embarrassing**.

As per standard startup-style-execution, you don't fix it until it is broken. Well, this crossed the "broken" threshold in my book. The question was what to fix: the website uptime monitoring or the disk layout. Disk layout would take more effort and would be targeted to handle just 1 situation, therefore improving the monitoring seemed the prudent choice.

# SMS Alerts Aren't Enough
I typically sleep with my phone within earshot and have site24x7 send me TXT messages (actually just emails to the phone's email address XXX@mobile.att.net), but I had been helping out a friend that weekend on an SMS solution, so I was tone-deaf to the hundreds of test SMS's floating around. This resulted in the hard requirement of a phone call in the event of an outage. So after many searches (as there are A LOT of website monitoring services), I settled on trying out just a few. I was surprised that most of the services out there had really crappy UI's and antiquated visual designs. I eliminated most of these on the quasi-objective basis that they were out of touch or not thriving as a business. This narrowed the field considerably.

## Binary Canary
<img src="/images/logo-binarycanary.jpg" alt="binary canary logo">

I won't bore you with the details, but I settled on [Binary Canary](https://binarycanary.com/). Their UI was current, they had phone call escalation and the design of the escalation profiles and management was pretty good. Now when the site is down, I've got a nice escalation progression from email to SMS to calls to my cell phone and finally calls to my home phone (yes I know I'll get hell from my wife at some point for this).

## Pricing
Binary Canary is more than site24x7 (which I still have running on non-critical sites), but $20/month is reasonable for current, critical projects. You then buy "Notification Credits" at $0.25 per credit as they have real costs for SMS and phone calls. SMS via email is free.

## Phone Call Escalation Experience
<img src="/images/escalation.png" alt="binary canary escalation"/>

I've tested the phone call experience a couple times so far. During planned server upgrades, I have let the alerts go through their escalation and it has worked well. The phone call is an IVR that explains who they are. If you enter in your PIN, it will give you the details of which monitor has tripped and allow you to continue the alert escalation path or acknowledge this alert and stop notifications.


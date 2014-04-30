---
layout: post
title: mongoDB fails for website analytics use case
date: 2010/04/04
---

Update: [Now Trying Amazon's SimpleDB](/2010/04/05/amazon-aws-simpledb-for-website-analytics)

When I begin a web project that will need clickstream analysis, I typically start with just dumping the requests into a 'page_views' table and maybe a 'session_logs' table. I include almost all the data possible to record for that one request and also do some post-processing via a batch job for some meta-data, e.g. a flag on the session for 'is_entry_page', 'is_exit_page', 'referral_type' (i.e. Google SEO, link from partner, etc.) so I can do some slicing and dicing in reports.

For some projects, I even capture the copious amount of robot page views. "CRAZY!" you shout? Well, yes, but sometimes I'm just too lazy to exclude it. Other times I'm curious about who is curious about my site. Other times I really need the data to answer whether Googlebot gotten to a specific set of pages and how often is it coming back?

Obviously this breaks down at some point, but it is always a fast and useful implementation and some projects don't get to the point where I need to clean it up. Well, one such project is at the cleanup stage now, so I've been evaluating technologies besides MySQL, "NoSQL" as they seem to be called these days, i.e. Redis, Cassandra, and the like. The reason why I'm ditching MySQL for this is that it is too expensive whenever I want to add a new column to the tables for meta data, e.g. a flag for sessions that used my internal search. This is a common problem that many people have blogged about, so nothing new there. Yes, there are ways around it, e.g. using separate tables as indexes. [FriendFeed seems to take this to an extreme and interesting degree](http://bret.appspot.com/entry/how-friendfeed-uses-mysql)

But I confess that the primary reason I'm doing a NoSQL is I've never developed and deployed one in production before (Memcached doesn't count).

So [mongoDB](http://www.mongodb.org/) looked the most promising, primarily because of its flexible way of doing queries. I also liked the feel of its interactive shell. So I started a side project to figure out the most efficient way to construct and migrate my 3 million rows of 'page_views' into mongoDB. I started out using the mongo shell, then graduated to the raw Ruby driver, then finally to [MongoMapper](http://github.com/jnunemaker/mongomapper/) Starting from the ground floor better enables me to learn what's underneath rather than treat it as a black box.

Well, the new mongoDB schema, migration and website slicing/dicing was a success. The reports are _WICKED FAST_, but there was one little caveat that broke the whole system:

[http://www.mongodb.org/display/DOCS/Indexing+Advice+and+FAQ](http://www.mongodb.org/display/DOCS/Indexing+Advice+and+FAQ)

<blockquote>
<b>Make sure your indexes can fit in RAM.</b>
If your queries seem sluggish, you should verify that your indexes are small enough to fit in RAM. For instance, if you're running on 4GB RAM and you have 3GB of indexes, then your indexes probably aren't fitting in RAM. You may need to add RAM and/or verify that all the indexes you've created are actually being used.
</blockquote>

After importing the data and specifying indexes on all the fields I pivot on, I checked the size of my index for that collection:
    > db.report_sessions.totalIndexSize()                              
    36782080
_Yikes!!!_ That's 36MB of indexes for 30,000 documents. Traffic is steadily growing, so I'll be at 10x that in no time and 100x soon after. Requiring 3GB of RAM just for my indexes on a small site doesn't scale. I wonder if I'm doing something wrong. 

I'm going to try Amazon's SimpleDB next, although I'm sure it will be a turtle compared to a local mongoDB install. Also, I wonder how fast the reads will be since I'm currently [hosting at Slicehost](http://www.slicehost.com/).


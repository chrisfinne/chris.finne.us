---
layout: post
title: Solr JAVA on Mac
date: 2010/05/29
---

I'm in the process of migrating from Sphinx to Solr.
Sphinx does a nice job of geosearch out of the box, but it is a VERY basic approach that can't scale to a large number of records, e.g. 22 million.
My production servers are Linux, specifically [Ubuntu AMI's from alestic](http://alestic.com/).
When I do test indexing of my 22 million records, everything is smooth as butter on Ubuntu, 
but I have all sorts of weird problems on my new Mac Pro - Snow Leopard. I just don't think Java runs well.

< update >

Apple released a Software Update that upgraded Java recently and it seems to have improved my Solr stability, but when I import the 22 million records
into Solr with a [Ruby Enterprise-REE](http://www.rubyenterpriseedition.com/) I get some weird behavior where the processes break like the Rails
debugger has been triggered and I have to manually continue the processes. I then tried JRuby 1.5 installed via [RVM](http://rvm.beginrescueend.com/)
and the indexing was perfectly stable (and I could do real threads!)

< another update >

Funky (probably invalid characters) are causing this problem. I scraped some webpages that either aren't encoded in UTF-8, but I'm treating them as such. Not a major problem for me, so never got around to fixing it. Rails 3 and Ruby 1.9 handle all this much more elegantly now, so next time around this block I'll pay attention to the encoding of my Strings.

### Reminders for when importing the 22 million records:
- When in Development, turn off Rails logging
<code>
    development.rb:
      config.log_level = :error
</code>
- Don't do commit's to SOLR as it will make it launch a new searcher. Do the commit and optimize after all the records have been imported.

### Setting up large Solr instance on EC2
It was taking over a minute to index 1000 documents, even using jruby 1.5, which is faster for converting the ActiveRecord objects into hashes. 
I figured it was the slow EC2 disks. Even though I setup RAID0 on 8 EBS volumes, the disk performance was still ridiculously slow compared to my Mac.

So I upgraded from my EC2 instance from m1.large (7.5GB RAM, 2 cores w/ 2 EC2 Compute Units each) to m2.2xlarge (34.2GB RAM, 4 cores w/ 3.25 EC2 Compute Units each)
and planned to put the index on a RAM disk (automatically available on Ubutnu at /dev/shm). Well, this upgrade of CPU and going to the RAM disk
was very disappointing. I only when from about 60-70 seconds down to 30-35 seconds, so it looks like there is still a significant bottleneck by not
having MySQL local.

< update >

Now I have it on a Cluster Compute Instance (cc1.4xlarge).


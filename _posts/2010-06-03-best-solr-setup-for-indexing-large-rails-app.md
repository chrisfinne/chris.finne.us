---
layout: post
title: Best Solr Setup for Indexing Large Rails App
date: 2010/06/03
---

### Background on my Solr / Rails effort
I've been doing a lot of importing recently as I'm trying to find the fastest way to get my 22 million records into Solr. 
I've used DataImportHandler and it is of course by far the fastest, but I need to massage some of the data in Ruby first, so DIH is out.
I use [Matt Mitchell's](http://github.com/mwmitchell) [rsolr](http://github.com/mwmitchell/rsolr) libraries. 
I've looked at [SunSpot](http://outoftime.github.com/sunspot/) and other Solr libraries for Rails, but this time I want to start
from a pretty low level as I've got a lot of docs and don't want any unnecessary overhead that some of these higher-level libraries
might add.


### rsolr-direct vs. normal rsolr/jetty setup

These are rough numbers, but thought you might be interested

on Snow Leopard jruby 1.5.0rc3 vs. REE 1.8.7:

- For basic ruby stuff (generating the hashes to send to Solr): <b>jruby is roughly 20-25% faster</b>
- rsolr-direct vs. normal rsolr-ext to a locally running jetty, <b>direct is 5x to 10x faster</b>.

But this is just a single-process, single-threaded test, so not very real-world in most cases.

### Factors and Permutations
There are so many factors that determine where the bottleneck and what the final performance will be:

- jruby vs ree (haven't tried ruby 1.9 yet)
- rsolr-direct vs. rsolr=>jetty
- single solr core vs. multi-core
- single CPU core vs multiple processors/cores
- threads vs processes
- size of the doc being indexed
- fields stored vs. not stored

Here are just a few of the permutations that I've tried:

- Single Solr Core, JRuby, rsolr-direct, multiple jruby threads with a mutex around rsolr-direct writes
- Single Solr Core, multiple REE processes, rsolr-ext, solr on jetty
- Multiple Solr Cores, multiple JRuby processes using rsolr-direct, then doing a merge of the Solr cores

The last one is what I'm starting to lean towards because it makes the best use of my multiple cores on my dev box and will likely do the best on a super big EC2 instance. My Mac has 8GB of DDR3 RAM and a 2.9GHz quad-core processor and it seems that CPU is the limiting factor. I tried 3 JRuby processes, but the CPU was only at about 50%. I'm now doing 7 JRuby processes and the CPU is running at about 90%.


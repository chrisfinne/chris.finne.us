---
layout: post
title: Improving Sphinx Geo Query Performance on Amazon EC2
date: 2010/05/04
---

So I recently did an [migration from Slicehost to Amazon AWS](/2010/04/18/migrating-from-slicehost-to-amazon-aws-ec2/). My website does a lot of intense geo queries with [Sphinx](http://sphinxsearch.com/) and they are pretty slow. It takes up to a few seconds to do the query the load all the objects from SQL. 

With Google getting more aggressive at taking [site speed](http://www.mattcutts.com/blog/site-speed/) into considering in their search algorithm, I wanted to significantly improve the performance of my website. Since 99.999% of the pages on our site use a geo-query, it seems like the perfect place to optimize.

### Setup RAID 0 on EBS volumes with XFS - No Improvement

So I [found](http://orion.heroku.com/past/2009/7/29/io_performance_on_ebs/) a [few](http://www.mysqlperformanceblog.com/2009/08/06/ec2ebs-single-and-raid-volumes-io-bencmark/) [articles](http://initsix.co.uk/content/how-make-xfs-4-disk-raid-0-partition-ec2-mysql-maximum-disk-io-performance) on improving disk performance, specifically reads, by striping EBS volumes, I figured I'd give it a shot. I created 8 20GB volumes and set them up with RAID 0.

<code>
mdadm --create /dev/md0 --metadata=1.1 --level=0 -n 8 --chunk=256 /dev/sdf /dev/sdg /dev/sdh /dev/sdi /dev/sdj /dev/sdk /dev/sdl /dev/sdm

mkfs.xfs /dev/md0

mkdir /mnt/raid0

mount /dev/md0 /mnt/raid0
</code>

and then moved my 3.4GB sphinx index to that partition. Some quick benchmarks didn't show any material improvement in the geo-queries. So while this was an interesting little project, it didn't yield significant results here. 

Although, indexing of the data when from 28 minutes down to 24 minutes, so that 15% improvement is nice to have, especially since I'm about to go to 10x the data soon.

### Sphinx max_matches - Big Improvement
I had my max_matches set to 50,000. This is really high, but initially for a reason. I have millions of records and I wanted Googlebot to be able to page through them. Well, Sphinx has a really lame limitation about pagination. It is tied to the max_matches parameter, So I set it at 50k initially to get Google off and running. Since then, the navigation structure of the site has broadened and I've added [sitemaps](http://www.sitemaps.org/) so I no longer need that large 50,000 limit for pagination. Anyhow, Googlebot got past that number in the pagination recently and things blow up, so I now just catch the Exception and 301 redirect to the home page in that case so that Googlebot eventually gives up on those high number pages.

On my macbook pro, I cranked the 50,000 down to 5,000 and got an immediate 30% performance gain. On production it worked out to about 35%. Note that you can set the :max_matches per query in Sphinx and [the Rails plugin ThinkingSphinx](http://freelancing-god.github.com/ts/en/) or do the global setting on the sphinx config file and thinking-sphinx sphinx.yml file.





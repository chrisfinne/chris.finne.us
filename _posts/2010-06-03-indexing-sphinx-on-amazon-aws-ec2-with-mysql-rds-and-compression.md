---
layout: post
title: Indexing Sphinx on Amazon AWS EC2 With MySQL RDS and Compression
date: 2010/06/03
---

I'm using [Amazon's Relational Database Service](http://aws.amazon.com/rds/) which runs MySQL 5.1 on a Large EC2 instance for me.

I have over 2 million records in a Rails application that I index with Sphinx.

Now I'm looking at MySQL Compression.
Sphinx can use it with this switch from the [Sphinx Docs](http://sphinxsearch.com/docs/current.html)

<code>
mysql_connect_flags = 32 # enable compression
</code>

### Sphinx Indexing (to a RAM disk) w/o compression
<code>
real	15m17.159s<br/>
user	14m20.330s<br/>
sys	0m28.630s
</code>

### Sphinx Indexing (to a RAM disk) w/ MySQL Compression:
<code>
real	15m45.433s<br/>
user	14m58.080s<br/>
sys	0m13.310s
</code>

So no material difference between the two.

As an aside:

### Sphinx Indexing (to an EBS boot disk) w/ MySQL Compression:
<code>
real	16m0.305s<br/>
user	14m56.730s<br/>
sys	0m16.230s
</code>

so it looks like using the RAM disk on EC2 doesn't affect indexing performance all that much.

### MySQL Compression for Rails
In parallel I was also looking up [how to enable MySQL compression in a Rails app using the MySQL gem](/2010/06/03/how-to-enable-mysql-compression-with-rails-and-the-mysql-gem/)



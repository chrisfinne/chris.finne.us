---
layout: post
title: How to Enable MySQL Compression With Rails and the MySQL Gem
date: 2010/06/03
---

While looking to optimize my Rails performance on Amazon EC2 and RDS, I wanted to experiment to see if there are any
performance improvements with enabling MySQL compression in the connection protocol since with RDS, the MySQL instance
is running on another machine and I'm unsure how far away that machine really is. (Spoiler: it seems to worsen performance with EC2 and RDS)

A couple years ago, I wanted to use MySQL Stored Procedures with a Rails app and found that Rails didn't support it.
The default MySQL connection from ActiveRecord didn't have the proper flag (Mysql::CLIENT_MULTI_RESULTS) enabled to deal with the results from a stored procedure.
This is discussion is currently captured in [Lighthouse](https://rails.lighthouseapp.com/projects/8994-ruby-on-rails/tickets/3151)

It looks like the flag (Mysql::CLIENT_MULTI_RESULTS) is now enabled by default in Rails 2.3.5 (maybe a bit earlier)

If you look in: __activerecord/lib/active_record/connection_adapters/mysql_adapter.rb__ 

__ActiveRecord::Base__ class method __mysql_connection__
about line #73

<code>
default_flags = Mysql.const_defined?(:CLIENT_MULTI_RESULTS) ? Mysql::CLIENT_MULTI_RESULTS : 0
</code>

Do a monkey patch and bitwise OR the flags to change it to:

<code>
default_flags = Mysql::CLIENT_MULTI_RESULTS | Mysql::CLIENT_COMPRESS
</code>

(assuming your MySQL gem is installed and supports both flags like the MySQL 2.8.1 gem does)

I then did somewhat unscientific comparision with and without the flag

<code>
./script/runner 'puts Benchmark.realtime{Business.all(:limit=>10000)}.to_s'
</code>

### Results
With the patch, the test typically took 1.6 seconds. Without the patch it typically took 0.6 seconds. So it looks like __the added compression
adds more overhead on the CPU than what you gain in network savings__ when dealing with EC2 and RDS in the same availability zone.

So as I saw with [MySQL compression enabled in my Sphinx setup](/2010/06/03/indexing-sphinx-on-amazon-aws-ec2-with-mysql-rds-and-compression/)
the network bandwidth between RDS and EC2 instances isn't the bottleneck I'm looking for.

<button href="#" class="sirenGo">TXT ME</button>

<script>
(function() {
 var sirenApiKey='8491b3915a21448c9aa3a3f07846123b';
 var sirenCssClass='sirenGo';

 var script = document.createElement('script');
 script.type = 'text/javascript';
 script.async = true;
 script.src = 'http://conversations-staging.aerialink.net/siren.js';
 script.id = "siren_script_id";
 script.setAttribute('sirenapikey',sirenApiKey);
 script.setAttribute('sirencssclass',sirenCssClass);
 (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(script);
})();
</script>

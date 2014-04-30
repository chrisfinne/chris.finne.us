---
layout: post
title: Migrating From Slicehost to Amazon AWS EC2
date: 2010/04/18
---

I love [Slicehost](http://www.slicehost.com/) and they are my default go-to hosting provider for projects and personal stuff. Sometimes I'll start a project on Amazon's EC2 if there is a decent chance that the project will have to scale within the next year. [Heroku](http://www.heroku.com/) is pretty cool and I host this blog on them, but I'm a UNIX guy and I just like having SSH and full root access to my systems.

I've been choosing Slicehost because:

- automatic backups
- nice web console for the sysadmin
-  cheap
-  good support
-  [great articles](http://articles.slicehost.com/2008/4/25/ubuntu-hardy-setup-page-1) published in their forum for basic setups

So my current startup has been going for about a year now and I had a few slices on Slicehost, the DB server being on a 2GB RAM slice. It was way too slow for the size of DB. I have probably around 8 million rows in the DB and when I had to make some schema changes to some of the larger tables, it meant an outage and lots of work.

### Database Digression
Yeah, I know Oracle handles this much better. I did plenty of work on v7 and a little on v6 and v8. But I like the price for MySQL. What about Postgres? Haven't really used it on a project yet. It reminds me of the first day at CS101 at Univ of Illinois. The professor handed out manuals for vi and emacs and said, "Pick one and learn it". I chose vi because the manual was smaller. That turned out to be the right choice because later in my career, I did a lot of work on customer systems and most of them didn't have emacs installed. Now I don't know if the manual for MySQL is smaller than the one for Postgres, but I chose it just because it seemed to be a bit more popular. I did dig deeper than that when evaluating, but it takes a lot of time to filter through the religious arguments, so I took the road most traveled and not sure if it made any difference.

Now back to the gist of the article...

### MySQL on Slicehost vs RDS
I could have upgraded to a larger slice, as Slicehost offers much larger ones, but they don't do automated backups on slices larger than 2GB RAM. Also, I've been itching to try out Amazon's new [Relational Database Service](http://aws.amazon.com/rds/) aka RDS.

RDS was pretty smooth to get running and against their recommendation, I just did a mysqldump of my DB and imported it that way even though it was over their 1GB recommended limit. It only took 30 or 40 minutes. Just make sure to turn off binary logging first by setting the backups to 0.

    rds-modify-db-instance my-db-name --backup-retention-period 0
    
also you should "--compress" the stream when doing the import:

    mysql -uuser -ppass -h your-db-hostname --compress my_db < mysqldump-file.sql


### Other features from AWS that I hadn't used before:

- Web console for EC2. Competitive with [Elasticfox, the FireFox plugin to manage EC2](https://addons.mozilla.org/en-US/firefox/addon/11626), but still a bit quirky.
- Performance monitoring of servers
- EC2 instances running off of EBS
- point-n-click cloning of an EC2 instance than runs off of EBS (but heads-up, it shuts down the instance!!!)

## Problems with the migration to AWS

#### Elastic IP

I did have problems with Elastic IP addresses this time around. I had one and it was working fine, but when I launched a new EC2 instance and changed the IP over to that, the association kept dropping. Then it would associate with either slice for more than a few minutes. I shut that one down and started up another Elastic IP and had the same problem. 

#### Point-n-Click Create EBS Volume Image
It immediately shuts down the image that it is cloning. Probably should have guessed that, but it caught me by surprise.

#### Sending Email to Port 25 Got Blocked
My app sends out emails to customers. It also sends a lot of emails via [ExceptionLogger](http://github.com/rails/exception_notification) and [God](http://god.rubyforge.org/) (when I say "a lot" I mean "a lot". probably an indication of how lame of a programmer I am ;-) Well, just a few hours into the migration, I got an email from "Amazon EC2 Abuse" saying:

*Dear EC2 Customer,
You recently reached a limit on the volume of email you were able to send out of SMTP port 25 on your instance:*

Well, they had a nice link for me to register my use case with them, which I did. This is all understandable, but annoying none-the-less.

Anyhow, I relay my emails through [Rackspace Email](http://www.rackspace.com/apps/email_hosting) with [Postfix](http://www.postfix.org/) after I [DKIM](http://www.dkim.org/). I also have my [SPF](http://en.wikipedia.org/wiki/Sender_Policy_Framework) setup, so my emails are pretty solid. I just changed my relay to contact Rackspace's SMTP server on a different port, 587 I think.

## Conclusion
The migration went very smoothly and Amazon has made a lot of progress with their offering in the past 16 months since I last setup an infrastructure on them. 

- The RDS instance on an m1.large is *blazing fast* and I no longer worry about schema changes to large tables.
- The EBS volumes for the root partition is a great advancement as well as their performance is better, snapshots are easy as is cloning.
- Used a [Ubuntu image with EBS Boot from alestic](http://alestic.com/)
- Web Console past due. I look forward to seeing the rest of the offerings, like SDB, on the console
- Performance Monitoring is nice
- Haven't played with Load Balancing yet, but look forward to it.

Amazon is the 600lb gorilla in the cloud space. I wonder why anyone else even tries.


---
layout: post
title: Amazon AWS SimpleDB for Website Analytics
date: 2010/04/05
---

So I [tried mongoDB](/2010/04/04/mongodb-fails-for-website-analytics-use-case), but the math didn't add up. I'm now doing the math for using AWS's SimpleDB for website analytics.

### Some SimpleDB Limits
from the [dev guide](http://docs.amazonwebservices.com/AmazonSimpleDB/2009-04-15/DeveloperGuide/)

- 256 attribute name-value pairs per item
- 1 MB request size
- 1 billion attributes per domain
- 10 GB of total user data storage per domain
- 25 item limit per BatchPutAttributes operation

### Some baseline numbers to work with
- One of my sites has about 10k sessions per day with an average of 2 page views per session.
- Use a single SimpleDB domain, one item per page view (or ajax call)
- Keep session meta-data in the "entry page" item, i.e. the first one of the session. 
- Session meta data is about 25 attributes
- A page view that isn't a session has up to 10 attributes

#### Math on Attributes Constraint
1 billion attributes / (10,000 sessions/day * 25 attributes) + (10,000 secondary page views/day * 10 attributes) = 2900 days = **8 years of data based on the attributes constraint**


Now with this project, I'm only up to 1600 sessions per day, but I expect the growth to continue FAR beyond 10k per day.



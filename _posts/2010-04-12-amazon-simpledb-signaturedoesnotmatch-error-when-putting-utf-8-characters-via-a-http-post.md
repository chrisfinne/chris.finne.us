---
layout: post
title: Amazon SimpleDB SignatureDoesNotMatch error when putting UTF-8 Characters via a HTTP POST
date: 2010/04/12
---

Ok, that title is a mouthful, but so was the bug. It was one of the more painful debugging sessions I've had in a while; probably a solid 10 hours.

I'm using [SimpleRecord](http://github.com/appoxy/simple_record) to access Amazon's SimpleDB from my Rails app. SimpleRecord sits on top of the [Appoxy's AWS gem](http://github.com/appoxy/aws) which is a fork of [RightScale's AWS gem](http://github.com/rightscale/right_aws)


I'm testing out importing all my web logs into SimpleDB and was getting this intermittent error: **Aws::AwsError: SignatureDoesNotMatch**

At first I narrowed it down to occasional UTF-8 errors and then read [this discussion thread](http://groups.google.com/group/simple-record/browse_thread/thread/3659e82491d03a2c#) and got a little more background. I did a bunch of trial-n-error with encoding, double-encoding, not encoding, etc. to determine a pattern of what, when and where it was failing.

Here's the general matrix for when it succeeded/failed:
    Success: PutAttributes w/ no UTF-8 characters with an HTTP GET
    Success: PutAttributes w/ UTF-8 characters with an HTTP GET
    Success: PutAttributes w/ no UTF-8 characters with an HTTP POST
    Failure: PutAttributes w/ UTF-8 characters with an HTTP POST

So I even suspected Ruby's HTTP library that it was some how messing with the body of the POST, so I turned on Apache's DumpIO module:
    DumpIOInput On
    DumpIOLogLevel notice
and had the AWS library do the GET's and POST to my localhost so I could watch what was happening. Everything looked fine there.

By this time, my eyes were seeing double. So I started down the line of investigation: *"Other languages must have run into this. Let me look at their code..."*

**Voila** the first article I read was this [PHP example](http://thetalecrafter.wordpress.com/2009/02/19/accessing-aws-simpledb-from-php/). It is using the content type of:
    'application/x-www-form-urlencoded; charset=utf-8'
rather than:
    'application/x-www-form-urlencoded'
that the AWS gem is using and the [Amazon SimpleDB API](http://docs.amazonwebservices.com/AmazonSimpleDB/2007-11-07/DeveloperGuide/index.html?REST_RESTAuth.html) specifies. That made so much sense that you could say it really tied the room together.


## Solution / Patch
So the 1 line solution is in right_awsbase.rb line # 398

    request['Content-Type'] = 'application/x-www-form-urlencoded'
should be:
    request['Content-Type'] = 'application/x-www-form-urlencoded; charset=utf-8'

DONE!





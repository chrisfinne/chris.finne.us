---
layout: post
title: Multi Server Thinking Sphinx
date: 2010/04/09
---

Ok, so I came across a [Schroedinbug](http://en.wikipedia.org/wiki/Unusual_software_bug) this morning. I added a second server a few weeks ago to run Jobs via [DelayedJob](http://github.com/collectiveidea/delayed_job) as they were eating up too many resources for the [Passenger](http://www.modrails.com/) / DB / [Sphinx](http://sphinxsearch.com/) server. Today is the first day that I noticed that the jobs were not able to do searches.

<pre>
  >> User.search 'asdf'
  Riddle::ConnectionError: Connection to 127.0.0.1 on 9312 failed. Connection refused - connect(2)
</pre>

## Problems
1. iptables was blocking access from the jobs server to the sphinx port on the search server
2. sphinx was listening on 127.0.0.1 and not 0.0.0.0
3. thinking-sphinx was configured to connect to 127.0.0.1 on all servers

## Resolution
1. Setup /etc/hosts files on both servers to point the hostname alias "sphinx" to the app/search/db server (this way i don't need different config files for the 2 servers)
2. ensure ThinkingSphinx config is updated for this hostname alias (rather than going to localhost)
3. Update your sphinx.conf (I roll my own sphinx.conf rather than let ThinkingSphinx generate one) to listen on the proper IP
4. Ensure your iptables allow connections from jobs server to search server

** Run on search server as root to make sure all traffic is accepted from jobs server **
<pre>
  iptables -A INPUT -s 10.1.2.3 -j ACCEPT
</pre>
I used to open just specific ports, but got tired of it, so just leave it wide open for server to server access. I also did the reciprocal command on the jobs server.

**config/sphinx.yml for Thinking Sphinx**
<pre>
production:
  port: 9312
  address: sphinx
</pre>

**sphinx.conf on Search Server**
<pre>
searchd
{
#  listen = 127.0.0.1:9312 # Remove me. Sphinx's default is: 0.0.0.0:9132
}
</pre>

## Result
Schroedinbug is gone. The cat died. No wait, or did it live? Does the cat represent the bug or the system working as it should? Arg, another problem to be solved today.


---
layout: post
title: Problems Making HTTPS Calls with Ruby on Windows
date: 2016/09/14
---

Assuming you are using [Ruby Installer](http://rubyinstaller.org/downloads/) to install Ruby on Windows, 
getting Ruby to make a call to an HTTPS server isn't straightforward.

<pre>
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\Administrator>ruby -v
ruby 2.2.4p230 (2015-12-16 revision 53155) [i386-mingw32]

C:\Users\Administrator>ruby -ropenssl -e 'puts OpenSSL::X509::DEFAULT_CERT_FILE'
C:/Users/Justin/Projects/knap-build/var/knapsack/software/x86-windows/openssl/1.0.1l/ssl/cert.pem

C:\Users\Administrator>
C:\Users\Administrator>ruby -ropenssl -ve 'puts Gem::VERSION, OpenSSL::OPENSSL_VERSION'
ruby 2.2.4p230 (2015-12-16 revision 53155) [i386-mingw32]
2.6.6
OpenSSL 1.0.1l 15 Jan 2015
</pre>

If you look at the default SSL cert file, you will notice that it is set to the path by the developer that packaged Ruby Installer.

So if you try to make an HTTPS call to a server, it will fail because there is no certificate chain to validate it against.

<pre>
C:\Users\Administrator> ruby -ropen-uri -e 'puts open("https://www.cloudflare.com/").read.size.to_s'
C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect': SSL_connect returned=1 errno=0 state=SSLv3 read server certificat
e B: certificate verify failed (OpenSSL::SSL::SSLError)
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `block in connect'
        from C:/Ruby22/lib/ruby/2.2.0/timeout.rb:73:in `timeout'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:863:in `do_start'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:852:in `start'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:318:in `open_http'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:736:in `buffer_open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:211:in `block in open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `catch'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:150:in `open_uri'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:716:in `open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:34:in `open'
        from -e:1:in `&lt;main>'
</pre>

You can download the standard [cURL cacert.pem](https://curl.haxx.se/ca/cacert.pem) and tell Ruby/OpenSSL where to find it by setting the SSL_CERT_FILE environment variable.

<pre>
C:\Users\Administrator>set SSL_CERT_FILE="c:/cacert.pem"

C:\Users\Administrator> ruby -ropen-uri -e 'puts open("https://www.cloudflare.com/").read.size.to_s'
C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect': SSL_connect returned=1 errno=0 state=SSLv3 read server certificat
e B: certificate verify failed (OpenSSL::SSL::SSLError)
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `block in connect'
        from C:/Ruby22/lib/ruby/2.2.0/timeout.rb:73:in `timeout'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:863:in `do_start'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:852:in `start'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:318:in `open_http'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:736:in `buffer_open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:211:in `block in open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `catch'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:150:in `open_uri'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:716:in `open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:34:in `open'
        from -e:1:in `&lt;main>'

</pre>

But that still won't work, even though you used the Ruby preferred forward slash for directory paths. Turns out you shouldn't use quotes... ARG!!!!


<pre>
C:\Users\Administrator>set SSL_CERT_FILE=c:/cacert.pem

C:\Users\Administrator> ruby -ropen-uri -e 'puts open("https://www.cloudflare.com/").read.size.to_s'
16085

</pre>

Ok, that's great, but does it work on Google?

<pre>
C:\Users\Administrator>ruby -ropen-uri -e 'puts open("https://www.google.com/recaptcha/api/siteverify").read.size.to_s'
C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect': SSL_connect returned=1 errno=0 state=SSLv3 read server certificat
e B: certificate verify failed (OpenSSL::SSL::SSLError)
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `block in connect'
        from C:/Ruby22/lib/ruby/2.2.0/timeout.rb:73:in `timeout'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:863:in `do_start'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:852:in `start'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:318:in `open_http'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:736:in `buffer_open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:211:in `block in open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `catch'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:209:in `open_loop'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:150:in `open_uri'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:716:in `open'
        from C:/Ruby22/lib/ruby/2.2.0/open-uri.rb:34:in `open'
        from -e:1:in `&lt;main>'

</pre>

Ok, why not? I tried various approaches like using stanard Ruby net/http library and forcing TLSv1_2, but that didn't help.

<pre>
irb(main):007:0> require "net/https"
=> true
irb(main):008:0> require "uri"
=> false
irb(main):009:0> uri = URI.parse("https://www.localwise.com/jobs")
=> #&lt;URI::HTTPS https://www.localwise.com/jobs>
irb(main):010:0> uri = URI.parse('https://www.google.com/recaptcha/api/siteverify')
=> #&lt;URI::HTTPS https://www.google.com/recaptcha/api/siteverify>
irb(main):011:0> http = Net::HTTP.new(uri.host, uri.port)
=> #&lt;Net::HTTP www.google.com:443 open=false>
irb(main):012:0> http.use_ssl = true
=> true
irb(main):013:0> http.ssl_version = :TLSv1_2
=> :TLSv1_2
irb(main):014:0> request = Net::HTTP::Get.new(uri.request_uri)
=> #&lt;Net::HTTP::Get GET>
irb(main):015:0> response = http.request(request)
OpenSSL::SSL::SSLError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `block in connect'
        from C:/Ruby22/lib/ruby/2.2.0/timeout.rb:73:in `timeout'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:923:in `connect'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:863:in `do_start'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:852:in `start'
        from C:/Ruby22/lib/ruby/2.2.0/net/http.rb:1375:in `request'
        from (irb):15
        from C:/Ruby22/bin/irb:11:in `&lt;main>'
</pre>

So I tried the popular [REST Client GEM](https://github.com/rest-client/rest-client)

<pre>
C:\Users\Administrator>gem install rest-client
Successfully installed rest-client-2.0.0-x86-mingw32
1 gem installed

C:\Users\Administrator>ruby -rrest-client -e 'puts RestClient.get("https://www.cloudflare.com/").size.to_s'
16085

</pre>

Ok, what is RestClient doing that standard library Ruby isn't?

I can even remove my SSL_CERT_FILE setting and RestClient still works

<pre>
C:\Users\Administrator>set SSL_CERT_FILE=

C:\Users\Administrator>ruby -rrest-client -e 'puts RestClient.get("https://www.cloudflare.com/").size.to_s'
16085

</pre>

It even works on Google


<pre>
C:\Users\Administrator>ruby -rrest-client -e 'puts RestClient.get("https://www.google.com/recaptcha/api/siteverify").size.to_s'
103

</pre>

Turns out RestClient is using some magic (apparently adapted from Puppet) to load the certificates from Windows...

[https://github.com/rest-client/rest-client/blob/master/lib/restclient/windows/root_certs.rb](https://github.com/rest-client/rest-client/blob/master/lib/restclient/windows/root_certs.rb)





---
layout: post
title: Quiet Assets Logging On Rails Webrick
date: 2017/07/12
---

Older Rails versions used to use the [Quiet Assets GEM](https://github.com/evrone/quiet_assets) which was then rolled directly into the Rails asset pipeline with


<pre>
# config/environments/development.rb

config.assets.quiet = true
</pre>

This switch works with Webrick and the development.log file, but I usually code with Webrick and byebug for debugging, so I have the console open and am always reading the STDOUT stream. The Asset logging stream is NOT silenced from the STDOUT so my console is still hopelessly cluttered.

Ruby monkey-patching to the rescue:

<pre>
if Rails.env.development?
  # Stop all the /assets/ logs in the webrick stdout
  module WEBrick
    class HTTPServer < ::WEBrick::GenericServer
      def access_log(config, req, res)
        # so assets don't log
      end
    end
  end

end
</pre>

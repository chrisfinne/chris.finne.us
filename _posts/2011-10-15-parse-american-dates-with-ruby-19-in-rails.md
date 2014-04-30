---
layout: post
title: Parse American Dates with Ruby 1.9 (in Rails)
date: 2011/10/15
---

<strong>UPDATE</strong>: This method no longer works as of Ruby 1.9.3. Try Jeremy Evans' GEM [ruby american date](https://github.com/jeremyevans/ruby-american_date) that works with all 1.9.x versions.

I probably spent days peeling the onion back to this solution, so had to share to avoid others from suffering the pain.

Ruby 1.8 happily parses American dates, but Ruby 1.9 refuses. When it comes to software, I'm not an ideologue. I just want to get the job done, so I won't pontificate, bloviate, opine or ramble on this topic. I'll find the workaround and move on with my life.

In a Rails project running on Ruby 1.9, I need to parse date and date/time strings that have the American format of month/day/year. I needed to do this in pure Ruby cases as well as assigning Date and Date/Time strings to ActiveRecord fields.

I tried parsing the strings before they got to ActiveRecord with stuff like:

<pre>
  Time.strptime(params[:start_at],"%m/%d/%Y %I:%M %p")
</pre>

But there are too many places to put this and this is a fragile and error-prone approach. I then found some monkey-patching solutions to work with just date strings [here](http://blog.mustmodify.com/2011/03/get-ruby-1-9-date-parse-to-assume-american-date-format.html
) and [here](https://gist.github.com/779859), but this still didn't solve the Date/Time strings.

I tried attacking the Date/Time string conversion in `ActiveRecord::ConnectionAdapters::Column` similar to [here](http://blog.nominet.org.uk/tech/2007/06/14/date-and-time-formating-issues-in-ruby-on-rails/) by overriding `string_to_time` and `fast_string_to_time`, but still had weird and inconsistent results when assigning a Date/Time string to an ActiveRecord attribute.

The final solution is of course a fraction of the code I cycled through and oh so simple, just swap the month and day if the string you are parsing looks like xx/xx/xxxx. I'm hooking in a a VERY low level, Ruby's `Date._parse()`

<pre>
  class Date
    class << self
      def _parse_with_american(str, comp=true)
        hash = _parse_without_american(str, comp)
        if str=~/\A(\d{1,2})\/(\d{1,2})\/(\d{4})/
          tmp = hash[:mon]
          hash[:mon] = hash[:mday]
          hash[:mday] = tmp
        end
        hash
      end
      alias_method_chain :_parse, :american
    end
  end
</pre>

I'm using [alias_method_chain](https://github.com/rails/rails/blob/13e00ce6064fd1ce143071e3531e65f64047b013/activesupport/lib/active_support/core_ext/module/aliasing.rb#L23) from Rails, but if you are not using Rails, well, you'll figure it out.

I have no idea why there is a `Date.parse` and a `Date._parse`, nor do I care at this point. I'm just glad to move on to solving real problems rather than wrestling with date/time strings.



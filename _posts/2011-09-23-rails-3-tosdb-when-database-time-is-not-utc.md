---
layout: post
title: Rails 3 to_s(:db) when database time is NOT UTC
date: 2011/09/23
---

I'm upgrading a Rails application from v2.x to v3.1 and the application stores its dates in MySQL in Pacific timezone rather than UTC.

I've set my config/application.rb with the following settings
<pre>
config.time_zone = 'Pacific Time (US & Canada)'
config.active_record.default_timezone = :local
</pre>

However, I just discovered that ActiveSupport time extensions don't really care about ActiveRecord's default_timezone setting.


<pre>
rails31>> Time.now.to_s(:db)
=> "2011-09-23 03:02:52"
rails31>> 1.second.ago.to_s(:db)
=> "2011-09-23 10:02:59"
</pre>

WHAT??? I know that these are 2 different classes:

<pre>
rails31>> Time.now.class
=> Time
rails31>> 1.second.ago.class
=> ActiveSupport::TimeWithZone
</pre>

This is because ActiveSupport sets to UTC before doing the "to_s" which makes sense with the default Rails configuration where all dates are stored in the DB as UTC.

[https://github.com/rails/rails/blob/master/activesupport/lib/active_support/time_with_zone.rb](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/time_with_zone.rb)

<pre>
# <tt>:db</tt> format outputs time in UTC; all others output time in local.
# Uses TimeWithZone's +strftime+, so <tt>%Z</tt> and <tt>%z</tt> work correctly.
def to_s(format = :default)
  if format == :db
    utc.to_s(format)
  elsif formatter = ::Time::DATE_FORMATS[format]
    formatter.respond_to?(:call) ? formatter.call(self).to_s : strftime(formatter)
  else
    "#{time.strftime("%Y-%m-%d %H:%M:%S")} #{formatted_offset(false, 'UTC')}" # mimicking Ruby 1.9 Time#to_s format
  end
end
alias_method :to_formatted_s, :to_s
</pre>

I guess I understand this because I'm dealing with 2 different libraries: ActiveRecord vs ActiveSupport, but it sure threw me for a loop in some of my tests where I would sometimes create values with Time.now and other times with 1.day.ago and the time-zone conversion would fail the test.

My solution to this issue is to start using .to_s(:localdb) rather than .to_s(:db)

<pre>
Time::DATE_FORMATS.merge!(:localdb=>"%Y-%m-%d %H:%M:%S")
</pre>


<pre>
rails31>> Time.now.to_s(:db)
=> "2011-09-23 02:56:34"
rails31>> Time.now.to_s(:localdb)
=> "2011-09-23 02:56:38"
rails31>> 1.second.ago.to_s(:db)
=> "2011-09-23 10:02:59"
rails31>> 1.second.ago.to_s(:localdb)
=> "2011-09-23 03:02:59"
</pre>

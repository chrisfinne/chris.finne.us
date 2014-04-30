---
layout: post
title: Ruby Rails Initialize BigDecimal with commas in the String
date: 2011/07/22
---

Came across a bug in the app while helping out on [SpotMe](http://www.getspotme.com/).

When you create a BigDecimal, it takes a String as the argument (not a Float or Fixnum)
<pre>
BigDecimal.new('99')
BigDecimal.new('44.23')
</pre>

In [SpotMe](http://www.getspotme.com/) when someone was entering an amount for a bill or a payment, the amount might have a comma in it. The problem is that BigDecimal ignores everything to the right of the first comma it encounters

<pre>
>> BigDecimal.new('1,200') == BigDecimal.new('1200')
=> false

>> BigDecimal.new('1,200') == BigDecimal.new('1')
=> true

</pre>

Monkey Patch to the rescue!

<script src="https://gist.github.com/1099504.js?file=gistfile1.rb"></script>


<pre>
>> BigDecimal.new('1,200') == BigDecimal.new('1200')
=> true

>> BigDecimal.new('1,200') == BigDecimal.new('1')
=> false

</pre>

While this concept can work for just Ruby, I used Rails' [alias_method_chain](http://weblog.rubyonrails.org/2006/4/26/new-in-rails-module-alias_method_chain). I've read some [arguments](http://yehudakatz.com/2009/03/06/alias_method_chain-in-models/) against this [approach](http://stackoverflow.com/questions/3689736/rails-3-alias-method-chain-still-used), but in the Startup world, I'm often just trying to get the job done fast, so I like to use it.

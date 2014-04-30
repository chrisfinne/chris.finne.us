---
layout: post
title: freeze - I can Modify CONSTANTS in Ruby
date: 2012/03/25
---

# STOP THE PRESSES!

I can modify a CONSTANT!

Whoa, that's a bit scary. I've been coding in Ruby for 5 or 6 years now. I probably knew this before and forgot. After all, when I was first learning Ruby, I read [Programming Ruby](http://www.ruby-doc.org/docs/ProgrammingRuby/) and this concept is likely in there (but i'm too lazy to confirm it right now). I know I'm making myself look like an idiot for going this long without fully internalizing (or forgetting) this, but oh well. I'm writing this for my own benefit. I won't forget this concept after spending a while explaining this important Ruby concept.

At least recently, I've been mentally assuming that I'd get an ugly warning when I'd __modify__ a constant:

<pre>
>> FOO='bar'
=> "bar"
>> FOO << '-more-bar'
=> "bar-more-bar"
</pre>

The warning I was expecting was this when I try to __redefine__ a constant:

<pre>
>> FOO='bar'
=> "bar"
>> FOO='not bar'
(irb):2: warning: already initialized constant FOO
=> "not bar"
</pre>

But technically the first example isn't __redefining__ it is __modifying__ the CONSTANT. Let's look at this again using object_id:

<pre>
>> FOO='bar'
=> "bar"
>> FOO.object_id
<strong>=> 2157596880</strong>
>> # Modify it
>> FOO << '-more-bar'
=> "bar-more-bar"
>> FOO.object_id
<strong>=> 2157596880</strong>
>> # Redefine it
>> FOO='not bar'
(irb):5: warning: already initialized constant FOO
=> "not bar"
>> FOO.object_id
<strong style="color:red">=> 2157574820</strong>
</pre>

Note that the object_id's changed when we __redefined__ but not when we modified. __Redefining__ is what triggers the CONSTANT warning.

But I recently did something like this:

<pre>
>> ARR=[1,2,3,4,5]
=> [1, 2, 3, 4, 5]
>> qwer=ARR
=> [1, 2, 3, 4, 5]
>> qwer.delete(3)
=> 3
</pre>

Then later on I used ARR again, and lo and behold it was missing the 3. __YIKES!!!__

Let's look at this again using `object_id`

<pre>
>> ARR=[1,2,3,4,5]
=> [1, 2, 3, 4, 5]
>> ARR.object_id
<strong>=> 2157583000</strong>
>> qwer=ARR
=> [1, 2, 3, 4, 5]
>> qwer.object_id
<strong>=> 2157583000</strong>
>> qwer.delete(3)
=> 3
</pre>

Ok, that makes sense. qwer is just another pointer to the same object.

# So what now?

How about both an offensive and defensive strategy. First the defense:

_If you don't want it modified, then __freeze__ it_

<pre>
>> ARR=[1,2,3,4,5].freeze
=> [1, 2, 3, 4, 5]
>> qwer=ARR
=> [1, 2, 3, 4, 5]
>> qwer.delete(3)
TypeError: can't modify frozen array
	from (irb):10:in `delete'
	from (irb):10
</pre>

And for the offense use __dup__ to create a duplicate object of the CONSTANT:

<pre>
>> ARR=[1,2,3,4,5].freeze
=> [1, 2, 3, 4, 5]
>> ARR.object_id
=> 2157590400
>> qwer=ARR.dup
=> [1, 2, 3, 4, 5]
>> qwer.object_id
=> 2157575680
>> qwer.delete(3)
=> 3
</pre>

__dup__ creates a duplicate object

# Great, but...

<pre>
>> FOO=42
=> 42
>> bar=FOO
=> 42
>> FOO.object_id
=> 85
>> bar.object_id
=> 85
>> bar += 99
=> 141
>> bar.object_id
=> 283
</pre>

Oh yeah, in Ruby, __Fixnum__ and __Float__ aren't ever modified, they are immutable. When you do operations on them you are just reassigning another __Fixnum__ or __Float__ to your variable, so __dup__ and __freeze__ are not needed.









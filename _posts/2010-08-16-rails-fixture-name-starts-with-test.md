---
layout: post
title: Rails Fixture Name Starts With Test
date: 2010/08/16
---

I'm using Rails 2.3.5 and I have a Rails model called "Testimonial". Whenever I run my tests, I get a funky error about ActionView::TestCase.

<code>
desktop:~/code/project chrisfinne $ ruby -Ilib:test test/unit/business_test.rb 
Loaded suite test/unit/business_test
Started
...E....
Finished in 0.311574 seconds.

  1) Error:
testimonials(ActionView::TestCase):
TypeError: wrong argument type Class (expected Module)
</code>    

I found this [Lighthouse ticket](https://rails.lighthouseapp.com/projects/8994/tickets/261-argumenterror-using-fixtures-named-test) that tells me that I can't have models called Test (which I already knew) and I cannot have any fixture files that _START with_ "test", e.g. "__test__imonials.yml". I didn't know that when I chose the name for my class. The fix is pretty easy...

1. Rename the fixture file from "test/fixtures/testimonials.yml" to something else like "test/fixtures/__a__testimonials.yml"
2. Associate the fixture with the proper class in test/test_helper.rb

<code>
  class ActiveSupport::TestCase
    ...
    set_fixture_class :atestimonials => Testimonial
    ...
  end
</code>

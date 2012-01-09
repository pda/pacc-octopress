---
layout: post
title: "MethodHuntingDelegator"
date: 2012-01-09 17:51
tags: ruby
---

I was implementing search in a Ruby app where the results objects were instances
of a mix of model classes. Each one had what could be considered a title and a
description, but the method names were inconsistent.

Wrapping each result in a `SearchResult` decorator to normalize the interface
seemed like a good idea. Ruby provides an abstract [Delegator][1] and a concrete
[SimpleDelegator][2] which gets most of the way there.

To normalize the method interface, I wrote an extension of `SimpleDelegator`
with a `#hunt_and_call(*candidates)` method, which finds and calls the first
method of the candidate list which the delegate responds to.

[1]: http://www.ruby-doc.org/stdlib-1.9.3/libdoc/delegate/rdoc/Delegator.html
[2]: http://www.ruby-doc.org/stdlib-1.9.3/libdoc/delegate/rdoc/SimpleDelegator.html

Here's the example calling code:

{% gist 1581535 example.rb %}

And the MethodHuntingDelegator implementation:

{% gist 1581535 method_hunting_delegator.rb %}

And of course:

{% gist 1581535 method_hunting_delegator_spec.rb %}


Completely insane? Useful enough for a gem? Better way of going about it? Give
me hell in [the gist comments](https://gist.github.com/1581535).

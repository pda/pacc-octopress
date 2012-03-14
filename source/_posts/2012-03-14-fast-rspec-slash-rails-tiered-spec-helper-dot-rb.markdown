---
layout: post
title: "Fast RSpec/Rails: tiered spec_helper.rb"
date: 2012-03-14 20:41
tags: ruby rails rspec
---

Slow Rails startup time is the TDD killer.

    paul@paulbookpro ~/project ⸩ time rspec spec/lib/method_hunting_delegator_spec.rb
    ..
    Finished in 0.00078 seconds
    2 examples, 0 failures
    rspec spec/lib/method_hunting_delegator_spec.rb -f d  6.76s user 1.64s system 91% cpu 9.225 total

Holy crap, that's 9 seconds of Rails startup, for 0.00078 seconds worth of
RSpec. And [this class/test][1] doesn't even use Rails! We can do better.

The culprit? That `require "spec_helper"` at the top of every spec file which
loads the entire of Rails:

    # This file is copied to spec/ when you run 'rails generate rspec:install'
    ENV["RAILS_ENV"] ||= 'test'
    require File.expand_path("../../config/environment", __FILE__)
    require 'rspec/rails'
    require 'rspec/autorun'
    # etc ...

There's a few ways to deal with this, each with their own [pitfalls][2]. After
trying many approaches, I've settled on a tiered RSpec initializer
(spec_helper.rb and friends) which I can choose when invoking RSpec.

All the spec files still `require "spec_helper"`, but it looks more like this:

{% gist 2034589 spec_helper.rb %}

Which means we can select different initializers using the SPEC environment
variable. The following `spec_helper_unit.rb` is perfect for the
method_hunting_delegator_spec which took 9 seconds earlier, because there's no
dependencies on Rails.

{% gist 2034589 spec_helper_unit.rb %}

The result?

    paul@paulbookpro ~/project ⸩ time SPEC=unit rspec spec/lib/method_hunting_delegator_spec.rb
    ..
    Finished in 0.00079 seconds
    2 examples, 0 failures
    SPEC=unit rspec spec/lib/method_hunting_delegator_spec.rb  0.81s user 0.08s system 99% cpu 0.890 total

Under a second (0.890) is much more like it, and we still get class autoloading
provided by ActiveSupport. I use this mode for just about everything except
subclasses of Rails components, and those I keep a slim as possible. Moving
logic into [SOLID][3] classes is something you'll benefit from anyway, and
these faster tests provide extra incentive. This example was a spec for a
standalone class living in `RAILS_ROOT/lib/` but I use it for all sorts of
classes under `app/models/`, `app/presenters/`, `app/forms/` etc.

But this zero-Rails initializer doesn't help with testing your ORM-subclasses
(we'll begrudgingly call them "models") which depend on ActiveRecord:

    paul@paulbookpro ~/project ⸩ SPEC=unit rspec spec/models/book_spec.rb
    /Users/paul/project/app/models/book.rb:4:in `<top (required)>': uninitialized constant ActiveRecord (NameError)

And having tasted sub-second tests, 12 seconds is clearly unacceptable:

    paul@paulbookpro ~/project ⸩ time rspec spec/models/book_spec.rb
    .................
    Finished in 0.67016 seconds
    17 examples, 0 failures
    rspec spec/models/book_spec.rb  8.08s user 1.85s system 78% cpu 12.698 total

But if you write your classes carefully, they don't need to depend on much
from Rails except ActiveRecord. So let's write a spec_helper which loads &
configures ActiveRecord, plus a few other bits and pieces useful for testing
database-persisted models.

{% gist 2034589 spec_helper_model.rb %}

<em>You'll have to excuse the Devise hackery; it was the one component tightly
coupled into a model (`User`), and like most Rails app, that particular model
is at the center of the whole relationship graph. Perhaps there's a better
solution, but this got me fast model tests for all but the user_spec itself.</em>

Lets run that model spec again, this time boosted by `SPEC=model`:

    paul@paulbookpro ~/project ⸩ time SPEC=model rspec spec/models/book_spec.rb
    .................
    Finished in 0.58512 seconds
    17 examples, 0 failures
    SPEC=model rspec spec/models/book_spec.rb  6.50s user 0.21s system 98% cpu 6.844 total

Note that your model classes can still depend on external gems, but they'll
need to e.g. `require "money"` at the top. I suspect this explicit declaration
of dependency isn't a bad idea anyway.


Of course, there's always going to be specs which depend on the whole stack,
such as acceptance tests.  For those, here's the default spec_helper_full.rb;
basically like the original spec_helper.rb:

{% gist 2034589 spec_helper_full.rb %}

God speed.


[1]: http://paul.annesley.cc/2012/01/methodhuntingdelegator/
[2]: https://github.com/rspec/rspec-rails/issues/371
[3]: http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)

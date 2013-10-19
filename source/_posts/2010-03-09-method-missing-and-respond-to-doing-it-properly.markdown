---
layout: post
title: 'method_missing and respond_to?: doing it properly'
date: 2010-03-09
comments: true
permalink: /2010/03/methodmissing-and-respondto-doing-it.html
---

Sometime ago while using `method_missing` to implement some functionality I got the weird behavior that it would work only
*most* of the time but not always.  
In retrospect it's now pretty obvious but in the heat of the moment it took me about half a day of investigation and talking
before I figured it out.

What happened is that I did only half of the work.  
I defined `method_missing` but I forgot to define `respond_to?` accordingly.  
The result is that it worked when I called it directly on the instance, but failed if an association was involved.

To give an example, say you have a class like this:

``` ruby
class A < ActiveRecord::Base
  def example
    true
  end

  def method_missing(method, *args)
    if method.to_s =~ /example/
      example
    else
      super
    end
  end
end
```

Calling *example* directly on your instance works just fine.

``` irb
>> A.new.my_example
=> true

>> A.create!.example_me?
=> true
```

All fine, but as soon as you get an association in the middle of things:

``` ruby
class B < ActiveRecord::Base
  belongs_to :a
end
```

It just doesn't go well anymore:

``` irb
>> b = B.new(:a=>A.new)
=> #<B id: nil, a_id: nil, created_at: nil, updated_at: nil>


>> b.a.example?
NoMethodError: undefined method example?' for #<A id: nil, created_at: nil, updated_at: nil>
 from /var/lib/gems/1.8/gems/activerecord-2.3.5/lib/active_record/associations/association_proxy.rb:220:inmethod_missing’
from (irb):55

>> b = B.create!(:a=>A.create!)
=> #<B id: 4, a_id: 7, created_at: “2010-03-08 21:15:01”, updated_at: “2010-03-08 21:15:01”>
>> b.a.failing_example
NoMethodError: undefined method failing_example' for #<ActiveRecord::Associations::BelongsToAssociation:0xb70491d0>
 from /var/lib/gems/1.8/gems/activerecord-2.3.5/lib/active_record/associations/association_proxy.rb:220:inmethod_missing’
from (irb):57
```

Now, this last error is a bit clearer but I don't remember running into it at the time.  
If I had just followed the association_proxy:220 hint right away... ;)

What happens is that `b.a` doesn't return the instance but rather an [AssociationProxy](http://apidock.com/rails/ActiveRecord/Associations/AssociationProxy) instance that provides ActiveRecord's
extended functionality and this proxy relies on `A#respond_to?` to correctly forward method calls to the actual instance.

What I should have done is:<br>

``` ruby
class A < ActiveRecord::Base
  def example
    true
  end

  def method_missing(method, *args)
    if method.to_s =~ /example/
      example
    else
      super
    end
  end

  def respond_to?(method, include_private = false)
    if method.to_s =~ /example/
      true
    else
      super
    end
  end
end
```

``` irb
>> B.create!(:a=>A.create!).a.example?
=> true
>> B.new(:a=>A.new).a.failing_example
=> true
```

There are some much better write-ups on this topic, if you want to read more:

- [Using method_missing and respond_to? to create dynamic methods](http://technicalpickles.com/posts/using-method_missing-and-respond_to-to-create-dynamic-methods/)
- [Solving the method_missing/respond_to? problem](http://coderrr.wordpress.com/2008/07/11/solving-the-method_missing-respond_to-problem/)

Now, I must be honest here: what I was doing was a **big** code smell :)  
It taught me the lesson to use method_missing properly and was even quite fun to debug and all that but what I really needed and
end up doing in that case was a group of [delegates](http://apidock.com/rails/Module/delegate) here and there and voilà, it was all cool and clean.

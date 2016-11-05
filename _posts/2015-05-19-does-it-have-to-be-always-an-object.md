---
layout: post
section-type: post
title: Does it have to be always an object?
---

I am lucky enough to work at the company, where we are encouraged to self-develop and experiment. Our development process is not just barely delivering features, but a constant struggle to build better, more robust and easier to maintain system.

Quite quickly it became obvious that the original Rails' architecture would be not sufficient for us and that we need to introduce entities like presenters, serializers or service objects. We re-learned SOLID principles and we tried to apply the knowledge from a great book of Sandi Metz: ["Practical Object-Oriented Design in Ruby"](http://www.sandimetz.com/products).

During that time, we also learned the basics of Erlang and discovered functional programming, which honestly is more appealing to me. Lessons learned from this experience would be the subject of a separate post. They changed me as a programmer and stimulated me to question the way I write my Ruby code. One question  became particularly clear:

Do you always need an object with a state? Having an initializer means that you need to take care of an additional step and instantiate the object before performing any action. On purpose of an example let's take following service class:

```ruby
class UserService
  def initialize(user)
    @user = user
  end

  def give_points(points)
    @user.increment! :points, points
  end

  def facebook_notification(message, gateway: FacebookGateway.new)
    gateway.deliver_notification(@user.facebook_uid, message)
  end
end
```

Now, if we'd want to give points to the user, we'd call `UserService.new(user).give_points(100)`. To send a notification `UserService.new(user).facebook_notification("Hello")`. This makes total sense until you realize that in most cases you just need to do one call to single functionality from given service. In that case the instantiation step seems excessive.

One might say the two methods in fact belong to user model. This leads to the danger of having god objects in the system. We all have been there, right?

The alternative would be to create a module with a set of methods taking all required arguments to perform the action. Recently, at my current project we experiment with it and we get quite decent results. The example service could be rewritten in the following way:

```ruby
module UserService
  extend self
  def give_points(user:, points:)
    user.increment! :points, points
  end

  def facebook_notification(user:, message:, gateway: FacebookGateway.new)
    gateway.deliver_notification(user.facebook_uid, message)
  end
end
```

Then, aforementioned method calls would look as follows:

```ruby
UserService.give_points user: user, points: 100
UserService.facebook_notification user: user, message: "Hello"
```

After using this approach for a while I can say that it leads to the code which is:

- often much shorter
- easier to reason about (there is no hidden state of the object which influences the result of the function)
- easier to test
- easier to reuse

Of course, we still create classes if we feel it's more appropriate. However, I believe it's not necessary, because of the reasons listed above. There's [a great article](http://www.smashcompany.com/technology/object-oriented-programming-is-an-expensive-disaster-which-must-endhttp://www.smashcompany.com/technology/object-oriented-programming-is-an-expensive-disaster-which-must-end) out there, which lead me to start this experiment. It challenges Object Oriented Programming paradigm and lists most, if not all, its weaknesses. The excessive instantiation step is just one thing.

Ruby gives us the comfort of having the choice and this is great! Let me know what are your experiences with the functional approach in business code!
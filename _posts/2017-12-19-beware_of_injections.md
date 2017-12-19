---
layout: post
section-type: post
title: Beware of injections
tags: [ 'ruby' ]
---

Recently I found a sneaky bug in a Rails project, introduced by no one else than myself few months ago. The system applies CQRS and was supposed to compute the price of an order based on the requested items list and per-customer price tables, in which the  entries had a period of validity. From time to time we uploaded a new set of price entries to the price tables with fresh validity periods, but they were not visible for the system, despite being stored in the database table properly. Only restarting the app servers seemed to help. Note for my future self: caching of ActiveRecord queries does not take place between HTTP requests. Period.

After a longer while I’ve found that command handler class which looked similar to this:

```ruby
class CalculatePrice
  def initialize(time: Time.now.utc, get_price_table: GetPriceTable.new(time: time))
    @get_price_table = get_price_table
  end

  def call(command)
    price = @get_price_table.call(client_id: command.client_id, items: command.requested_items)

    # ...
  end
end
```

What happens here?

* the command handler uses a query to compute the price and passes the result to an aggregate
* I apply dependency injection to isolate the command handler in unit tests
* I also inject the current time to fix the time in integration tests

To get the whole picture you should know our commands are simple data structures (we use [Dry::Struct](http://dry-rb.org/gems/dry-struct/)). The infrastructure spawns the command handlers at boot. This means that the current time gets injected into the command handler only when the server starts. Bingo!

## How to fix it?
I must admit - I’ve been lazy. When writing my command handler class I have not considered its possible life-cycle. Everything would have been fine if only it would have been instantiated on every request, but I forgot to plan on a long living class. The question is when do I build the dependencies of a class? There are few possible paths from here:

### Factory outside
I can change my system to instantiate separate command handler for every request and the problem would be gone forever without touching my event handler. Simple, but in my opinion it hides the real problem. My command handler object should be as independent of the external circumstances as possible. The way how I create it should not impact its functionality. While it should be fine to create a tree of command handlers, the transient objects, like current time and objects depending on them, should be isolated and evaluated at the very last moment.

### Factory inside
Instead of passing the dependencies, I could pass recipes, how to build the dependencies, and then build them, when needed:

```ruby
class CalculatePrice
  def initialize(
    time: -> { Time.now.utc }, 
    get_price_table: -> { GetPriceTable.new(time: @time.call) }
  )
    @time = time
    @get_price_table = get_price_table
  end

  def call(command)
    price = get_price_table.call(client_id: command.client_id, items: command.requested_items)

    # ...
  end

  private

  # this could be extracted into a nice, meta-programming module
  def get_price_table
    @get_price_table.call
  end
end
```

So we replace the dependencies with factories.

### Outsource it!
Lastly, I would search for third-party libraries to manage my dependencies. Although Inversion of Control in Ruby may be a bit too much, I am keen to explore a bit. For example [dry-auto_inject](http://dry-rb.org/gems/dry-auto_inject/):

```ruby
module Quoting
    class Container
    extend Dry::Container::Mixin

    register "current_time" do
      Time.now.utc
    end

    register "queries.get_price_table" do
      GetPriceTable.new
    end
  end

  Import = Dry::AutoInject(Container)

  class CalculatePrice
    include Import['queries.get_price_table']

    def call(command)
      price = get_price_table.call(client_id: command.client_id, items: command.requested_items)

      # ...
    end
  end
end
```

With this setup, I can write unit tests:

```ruby
let(:handler) { CalculatePrice.new(get_price_table: -> { [] } }
```

However, I could not find a way to override the nested dependency when instantiating the command handler, what would make writing integration tests a challenge.

Also, dependencies are resolved early, when the object is instantiated. So this is not what we want.

## Conclusion
Today I will go with the second approach. It requires least messing with the current code base and feels PORO. It’s far from perfect, but, over time, I hope to extract the common behaviour to a module. Keep It Simple, Stupid.
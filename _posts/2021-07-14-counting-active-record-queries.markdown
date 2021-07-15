---
layout: post
title: Counting Active Record Queries [en]
description:
tags: [rails,ruby,activerecord]
comments: true
---

You can read the brazilian portuguese version of this post [here](http://thiagoa.github.io/contando-queries-do-active-record/).

Recently, some colleagues have been impressed with a code snippet I've written on a pull request to count Active Record queries. But why would one do that? Well, there are certainly good reasons!

I enjoy a TDD workflow when the situation calls for it, so I was trying to come up with a way to optimize the traversal of a tree of Active Record objects implemented  with the [`ancestry`](https://github.com/stefankroes/ancestry) gem. The problem is that GraphQL was accessing the tree roughly like this:

```ruby
# Pseudocode to explain the problem.
#
# The "children" collection is given by the ancestry gem.
publisher.root_nav.children do |child|
  # Recursively accesses the children of child until reaching max_depth...
end
```

As you can see, this code suffers from the N+1 problem where each children runs an additional query to fetch their children and so on. The first step to address the N+1 problem was to write some tests and then intercept the place where the root nav item was being fetched, so I started with the following code:

```ruby
class NavItemsQuery
  def self.call(root)
    root
  end
end
```

Then the GraphQL callsite turned from this:

```ruby
publisher.root_nav
```

Into this:

```ruby
NavItemsQuery.call(publisher.root_nav)
```

That of course did not change anything but I was off to a great start. At least I created the class where I would write my code, and my tests simply verified the contents of the tree until reaching `max_depth`.

The next step was to reuse the same tests but actually optimize the fetching strategy. In other words, the behavior would be kept the same but the code would be optimized. With a TDD workflow, how do I make that guarantee? One way is by counting SQL queries! With that in mind, I came up with the following test:

```ruby
it 'execute three queries or less' do
  root = create_tree(depth: 3)
  count = 0

  begin
    ActiveSupport::Notifications.subscribe('sql.active_record') { count += 1 }
    NavItemsQuery.call(root, max_depth: 3)

    expect(count).to be <= 3
  ensure
    ActiveSupport::Notifications.unsubscribe('sql.active_record')
  end
end
```

That seemed reasonable because my test exercised a tree with depth 3, and I after some analysis I figured out I could run one query for each level of the tree.

But how does counting queries work? Active Record has a pubsub mechanism where you can subscribe to `sql.active_record` to count any and all SQL queries being executed by Rails, and I simply leveraged the block's closure to count the queries. How great is that?

Since I had more than one test counting queries, I created a helper method:

```ruby
def counting_active_record_queries
  count = 0
  ActiveSupport::Notifications.subscribe('sql.active_record') { count += 1 }
  yield
  count
ensure
  ActiveSupport::Notifications.unsubscribe('sql.active_record')
end
```

And then I was able to turn that test into something very sweet:

```ruby
it 'execute three queries or less' do
  root = create_tree(depth: 3)

  count = counting_active_record_queries do
    NavItemsQuery.call(root, max_depth: 3)
  end

  expect(count).to be <= 3
end
```

Great! I also have to add that during the TDD session, the assertion changed into this:

```ruby
expect(count).to be 1
```

Because there was actually a way to fetch everything with a single query and further improve the performance, but I'll leave that up to another post :)

## Bonus: Debugging Active Record queries

Do you know that query you see in the Rails logs and you have no idea where it comes from, like a needle in a haystack? Well, you can use Active Support notifications to find it as well!

```ruby
ActiveSupport::Notifications.subscribe('sql.active_record') do |_, _, _, _, details|
  if details[:sql] =~ MY_QUERY_REGEX
    puts '*' * 50
    puts details[:sql]
    puts caller.join("\n")
    puts '*' * 50
  end
end
```

The above trick consists in:

1. Subscribing to Active Record queries;
2. The `details[:sql]` variable has the SQL query, which you can use to match against a portion of the query you are debugging;
3. You print the backtrace with `caller.join("\n")` to see where the query is being executed.

You can even slap a `binding.pry` inside the block you feel like it.

## Conclusion

This was a quick post but hopefully it's been useful! You can learn more about Active Support notifications [here](https://api.rubyonrails.org/classes/ActiveSupport/Notifications.html).

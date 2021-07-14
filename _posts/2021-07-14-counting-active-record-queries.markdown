---
layout: post
title: Counting Active Record Queries [en]
description:
tags: [rails,ruby,activerecord]
comments: true
---

Recently, some of my colleagues were impressed with a code snippet I've written down on a pull request. The goal was to count Active Record queries. But why would one do that? Well, there are certainly pretty good reasons!

I enjoy a TDD workflow when the situation calls for it; I was trying to come up with a way to optimize the extraction of an Active Record object tree with the `ancestry` gem. The problem is that GraphQL was accessing the tree roughly like this:

```ruby
root.children do |child|
  # Recursively access the children of child until reaching max_depth
end
```

As you can see, this code suffers from the N+1 problem where each children fetch their children and so on. The first step to address the N+1 problem was to intercept the place where the root nav item was being fetched. I started with the following code:

```ruby
class NavItemsQuery
  def self.call(root)
    root
  end
end
```

That of course did not change the behavior but I was off to a great start. At least I created the class where I would write my code, and my tests simply verified the contents of the tree until reaching `max_depth`.

The next step was to reuse the same tests but actually optimize the fetching strategy. With a TDD workflow, how do I make that guarantee? One way is by counting the Active Record queries! With that in mind, I wrote a test roughly like that:

```ruby
it 'execute three queries or less' do
  root = create_tree(depth: 3)
  count = 0

  begin
    ActiveSupport::Notifications.subscribe('sql.active_record') { count += 1 }
    NavItemsQuery.call(root)

    expect(count).to be <= 3
  ensure
    ActiveSupport::Notifications.unsubscribe('sql.active_record')
  end
end
```

That seemed reasonable because my test exercised a tree with depth 3 and I was able to make one query for each level of the tree. But how does that work? Active Record has a pubsub mechanism where you can subscribe to all SQL queries being executed, and I simply leveraged the block's closure to count the queries. How great is that?

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

And now I was able to rewrite that test:

```ruby
it 'execute three queries or less' do
  root = create_tree(depth: 3)

  count = counting_active_record_queries do
    NavItemsQuery.call(root, max_depth: 3)
  end

  expect(count).to be <= 3
end
```

Great! During the TDD process, I changed the assertion to:

```ruby
expect(count).to be 1
```

Because there was a way to fetch everything with a single query and great performance, but I'll leave that up to another post :)

## Bonus: Debugging Active Record queries

Do you know that query you see in the Rails logs and you have no idea where it comes from? You can use Active Support notifications to find it as well!

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

The trick consists in:

1. Subscribing to Active Record queries;
2. The `details[:sql]` variable has the SQL query, which you can use to match against a portion of the query you are debugging
3. You print the backtrace with `caller.join("\n")` to see where the query is being executed.

## Conclusion

This was a quick post but hopefully it's been useful! You can learn more about Active Support notifications in [this link](https://api.rubyonrails.org/classes/ActiveSupport/Notifications.html).

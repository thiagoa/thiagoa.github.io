---
layout: post
title: Some Notes on Testing and Pundit Testing
description: An example to illustrate some useful concepts on testing using the Pundit authorization gem.
tags: [ruby testing pundit]
comments: true
---

The intent of this post is to present some useful notes on testing. We will watch a close-to-real example
and discuss it step by step, inspecting the benefits, tradeoffs and possibly good practices.

This is also a post about Pundit, which is indirectly used to showcase the concepts.
We will use MiniTest and Mocha in the examples, respectively as the testing and mocking'n'stubbing
frameworks.

## About Pundit

Pundit is a simple and effective tool to implement authorization in your Rails app. 
What I like most about Pundit is that it doesn't impose restrictions on its use,
making the growth of your authorization logic healthy and flexible. You are free to implement your own
logic following OO best practices. It packs a helpful set of conventions for the use of authorization POROs – it
recommends that you adhere to simple naming rules and use a default structure for your policies, but you
aren't obliged to do so.

Pundit policies are usually instantiated with a *user* object as the first argument, and the main subject
of the policy as the second argument:

{% highlight ruby %}
policy = ArticlePolicy.new(user, article)
{% endhighlight %}

Pundit recommends that you create question mark methods for as many actions as you want to authorize in your
controller. In OO terms, your policy should **respond to** related actions in your controller. 
For instance, just ask the policy to check whether the user is allowed or not:

{% highlight ruby %}
# Should return a boolean
policy.publish?
{% endhighlight %}

In that case, your policy will send messages to the *user* and *article* collaborators to deliver a *yes* or *no*.

You can also omit the *article* collaborator (or whatever other object) in the controller action when you don't
need it.

## Unit tests

Pundit policies are very straightforward to test. If your requirements are simple, you can use
stubs to fake the required objects. Your stubs should be shallow and easy to setup:

{% highlight ruby %}
require 'mocha/mini_test'

class ArticlePolicyTest < MiniTest::Test
  def test_publisher_can_publish_articles
    user = stub(role: :publisher)
    article = stub(status: :draft)

    assert ArticlePolicy.new(user, article).publish?
  end

  def test_auditor_can_publish_articles
    user = stub(role: :auditor)
    article = stub(status: :draft)

    assert ArticlePolicy.new(user, article).publish?
  end

  def test_guest_cant_publish_articles
    user = stub(role: :guest)
    article = stub # Don't care about the status

    refute ArticlePolicy.new(user, article).publish?
  end
end
{% endhighlight %}

When you use stubs like that you must be able to smell trouble in advance. For example:
suppose you are checking in the *ArticlePolicy#publish?* method for a *:draft* status.
Your tests pass, but the real *article* object returns a *'draft'* string in response to the *status*
message. If so, comparing a string with a symbol fails. This is a typical situation of testing brittleness,
where your unit tests pass but the production code ocasionally fails. Debugging that is often boring and tiresome.

If you want convenience or need to query a complex set of relationships, you are good to
go with fixtures or factories, at the cost of having slower tests. If you use Factory Girl, you can use
the *build\_stubbed* feature. Here is an example of the same code snippet using fixtures:

{% highlight ruby %}
require 'test_helper'

class ArticlePolicyTest < ActiveSupport::TestCase
  def test_admin_can_publish_articles
    user = users(:admin)
    article = articles(:draft)

    assert ArticlePolicy.new(user, article).publish?
  end

  def test_auditor_can_publish_articles
    user = users(:auditor)
    article = articles(:draft)

    assert ArticlePolicy.new(user, article).publish?
  end

  def test_guest_cant_publish_articles
    user = users(:guest)
    article = articles(:draft)

    refute ArticlePolicy.new(user, article).publish?
  end
end
{% endhighlight %}

Some people might find this test repetitive, but I like its style. It's very readable, and clearly
tells what is being tested; you don't need to scroll up to acknowledge what the subject is. Replacing the
subject can be done with a simple search and replace in your text editor.

## Controller tests

Once you finish coding your unit tests, you are hopefully confident that your policy works. But the feature ain't
complete yet – you still need to test and implement the code in you controller. This is the kind of test you
might want to neglect, because the implementation is usually a one liner. I recommend that you also write
tests for that, because:

- You might forget to authorize your action.
- You might setup your policy in the wrong way; that happens specially when you are in a hurry.
  For instance, you could accidentally pass an *Article* class instead of an *article* object.
  The class is usually handled to the policy when you don't need to check for specific
  instance attributes, so the Pundit "policy finder" helper still knows which policy to look for.
- You want a safety net to watch out for breaks in your functionality, which may have business value.

This is what your controller code might look like, flash messages and other details omitted:

{% highlight ruby %}
class ArticlesController < ApplicationController
  def publish
    article = Article.find params[:id]

    # Pundit provides the authorize helper method, which creates and validates
    # the policy. It passes the current user as the first argument, assuming
    # your controller responds to the current_user method. It guesses
    # the right policy to use with a Railsy convention: because article
    # is an instance of Article it looks for ArticlePolicy. It automatically
    # throws NotAuthorizedError if the policy fails
    authorize article

    article.publish
    redirect_to article
  end
end
{% endhighlight %}

So, how do you test that? You might be tempted to write three controller tests, each corresponding to the three 
cases that exist in your unit tests. This is a bad choice, because:

- You are essentially testing the same thing in more than one place.
- You write more test code than you need – more code written is more code to maintain.
- Changing the requirements is costly, because you need to modify more than one test to make it
  work.

You can also restrict yourself to two tests: one to represent the case when some user is allowed, and another one to
represent when some user is denied. In that case you would pick up two examples from your unit tests and
implement them in your controller tests. The "allowed" test wouldn't explicitly exist, because it would
be implicit in the test method that exercises the main controller action.

Even with that approach you are still tying your particular test with an assumption that may change in the future.
What if now members can publish some kinds of articles? This isn't particularly a good example, but you get the idea.
You will have an extra failing test for free. You won't know it fails until you run the tests, specially if that code is
untouched for days or months.

## Implementing the controller test

Mocks are a good choice to test that situation, because:

- You already established a contract in your unit tests.
- You are confident that the policy works, and the unit tests cover all the possible situations.
- It is very unlikely that your policy signature will change.
- Your policy is a black box – your controller tests should know as little as possible about how it does its business

The real effect you have to test in your controller is whether it responds with a *403 forbidden* code, when the
policy fails. It doesn't matter how that happens internally: if it uses a *NotAuthorizedError* to generate
the HTTP response, or if the response code is set directly in your controller action.

You also need to check whether the right messages are sent to the policy, in the way your contract expects.

{% highlight ruby %}
require 'test_helper'
require 'mocha/mini_test'

class ArticlesController < ActionController::TestCase
  def test_publish_authorization
    article = articles(:draft)
    user = users(:admin) # Using admin here, but it really doesn't matter
    policy = stub(published?: false)
    ArticlePolicy.any_instance.stubs(:new).with(user, article).returns(policy)
    sign_in(user)

    patch :publish, id: article.id

    assert_response :forbidden
  end
end
{% endhighlight %}

This test is still very readable, and makes good use of mocks and stubs in my opinion. It also helps the reader
to see where the setup, the actions and the expectations are. The separation is enforced by the blank
lines between each part. Note that, however, that stub acts like a mock and establishes an expectation
on the contract of your policy. It is firstly part of the setup, but it also sets an expectation.
Of course you could split the setup phase into a *setup* method – provided by MiniTest or the testing
framework of your choice – if it made sense to do so.

With that kind of test you are free to change your policies without worrying if some other tests will
fail, and the code is much easier to maintain.

### Conclusion

This is the first post on my blog, and I'd love to know what you think. Here are my conclusions:

- Keep your tests readable. Make it easy to see on a glance where the setup, the actions and the
  expectations are.
- Some repetitions are beneficial to tests, specially if they increase the readability.
- Don't skip writing tests when the feature has business value.
- Try not to test for the same thing more than once, even if indirectly.
- Reflect about the benefits and tradeoffs of mocks and stubs, know when to use them.
- If you use stubs, make sure the stubbed objects are consistent with the real method names, arity and return
  types. We haven't talked about verified doubles in this post, but you should use them when appropriate.
- If aplicable, write tests that simulate inconsistent situations.

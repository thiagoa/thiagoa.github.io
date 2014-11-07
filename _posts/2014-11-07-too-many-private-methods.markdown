---
layout: post
title: Too many private methods!
description: An example of how too many private methods can lead to a better design
tags: [ruby refactoring]
comments: true
---

I don't like bloated views, I guess nobody does. Views can easily become the worst nightmare of
any application, if not properly taken care.

I was working on an application where there were lots of search forms spread throughout the views.
Those forms were not backed by ActiveRecord, so I had to use pure Rails form helpers,
like **text\_field\_tag**. If you use these kinds of helpers too much, any view can quickly
become a mess. 

I eventually got tired of building the forms that way, and felt the need to code a little
class which builds those forms in an elegant and straightforward manner.

My first step was to think about the form builder at a higher level. How would I like to build my form?

{% highlight erb %}
<%= search_form for: @model do |f| %>
  <%= f.text_field 'Search term', :by_term %>
  <%= f.select 'Status' :by_status, collection %>

  <%= f.submit 'Search' %>
<% end %>
{% endhighlight %}

That looks clean and good. The **by\_term** and **by\_status** parameters point to ActiveRecord scopes, related
to the **@model** object. There is custom code in the backend responsible for securely performing searches,
using those scopes - I will not show the code here for brevity purposes.

Besides building the form fields, here are some additional requirements:

- Each field must accept an **options** hash, just like the existing Rails form helpers.
- It must automatically fill in each field value after submit, gracefully keeping up with the form state.
- You are not required to pass a model, it is optional. The model can be used to help with building
  the form state, but the implementation must know how to search for the input's value attribute in
  other sources.

## Coding our form builder

Now we need to implement the code to make that interface work. We proceed by creating a Rails helper method
named **search\_form**:

{% highlight ruby %}
module FormHelper
  def search_form(options = {}, &block)
    form_options = options.delete(:form_options) || {}
    form_options[:method] ||= :get

    form_for :search_form, form_options do
      options[:context] ||= self
      SearchForm.new(options, &block)
    end
  end
end
{% endhighlight %}

This helper method builds our input fields, wrapping them inside a form tag.
As a next step we will build the *SearchForm* class:

{% highlight ruby %}
class SearchForm
  def initialize(**options)
    @view              = options.fetch(:context)
    @model             = options.fetch(:for, options[:model])
    @input_base_class  = options.fetch(:input_base_class,  'form-control')
    @submit_base_class = options.fetch(:submit_base_class, 'btn btn-default')

    yield self
  end

  def text_field(field_label, field_id, **options)
    options[:class] = css_class(options)

    field_value = find_value(options, field_id)
    field_html  = @view.text_field_tag(field_id, field_value, options)

    inline_labeled_field(field_html, field_label)
  end

  def select(field_label, field_id, collection, **options)
    options[:class] ||= css_class(options)

    value      = find_value(options, field_id)
    collection = @view.options_for_select(collection, value)
    field_html = @view.select_tag(field_id, collection, options)

    inline_labeled_field(field_html, field_label)
  end

  def submit(title, **options)
    options[:class] ||= @submit_base_class

    @view.submit_tag(title, options)
  end

  private

  def inline_labeled_field(field_html, field_label)
    return field_html if field_label.nil?

    @view.content_tag(:label) do
      "#{field_label}: #{field_html}".html_safe
    end
  end

  def css_class(options)
    [@input_base_class].push(options[:class]).compact.join(' ')
  end

  def find_value(options, field_id)
    options.delete(:value)         ||
    find_value_in_filter(field_id) ||
    find_value_in_request(field_id)
  end

  def find_value_in_filter(field_id)
    @model.try :get_filter_param, field_id
  end

  def find_value_in_request(field_id)
    @view.params[field_id]
  end
end
{% endhighlight %}

This is the code we need to make our form work, also taking into account our additional requirements.
I will not include the tests here, so we can keep our focus narrowed on the main subject

Note that we could have handled the main form tag there, instead of in the helper, but
that's OK for now.

## How it works

1. The constructor accepts a **context** option. That context will most likely be the **view context**,
   which gives us access to regular Rails helpers. Take a look at the **search\_form** helper method
   to see how we are handling the context.

2. The public methods are responsible for building the form inputs we need. They use raw Rails
   form helpers.

3. The methods **text_field** and **select** both find the input's value after submit. First 
   they try to find it in the **options** hash; if it's not there, they look into the
   model, using a **get_filter_param** method (I will not show it here).
   The only thing you need to understand is that the model keeps track of the last search performed,
   and accepts values from outside of the request params - if we wish to initialize the form with a default
   search we can, and the form still knows how to keep the state. Finally, if it doesn't find the value
   in the model, it looks for it in the request params.

4. Both methods return the result of **inline\_labeled\_field**, which itself returns a complete field together
   with its label. We only need the inline layout for now.

There are lots of things going on there. There are many designs to approach our problem, but this is the
simplest we could get at first.

## Too many private methods!

Notice we have 5 private methods. Too many private methods is a sign our class is doing too much, and
gives us a clue that we can abstract it even further. In Ruby we have the awesome feature of nested
classes. We can build these artifacts to lessen our burden and make our code cleaner and more organized. 

The **find\_value\_in\_filter** and **find\_value\_in\_request** methods could be condensed inside
**find\_value**, but we want to make our intent as clear as possible, effectively giving all
things a name to ease the life of people who read our code, and also for ourselves - so we prefer to keep
both methods.

But we can do even better: we can abstract the three methods in their own nested class inside
**SearchForm**, which we choose to name **FieldValueFinder**:

{% highlight ruby %}
class FieldValueFinder
  def initialize(model, view)
    @model, @view = model, view
  end

  def find(options, field_id)
    options.delete(:value)   ||
    find_in_filter(field_id) ||
    find_in_request(field_id)
  end

  private

  def find_in_filter(field_id)
    @model.try :get_filter_param, field_id
  end

  def find_in_request(field_id)
    @view.params[field_id]
  end
end
{% endhighlight %}

This class' responsibility is narrow and focused. We can easily understand what it does in its context,
and its dependencies are clear and explicit. It makes our **SearchForm** class easier to reason about, and
more compliant, although not completely, with the single responsibility principle (SRP). This is the
final **SearchForm** class, for now:

{% highlight ruby %}
class SearchForm
  def initialize(**options)
    @view              = options[:context]
    @input_base_class  = options.fetch(:input_base_class,  'form-control')
    @submit_base_class = options.fetch(:submit_base_class, 'btn btn-default')

    model         = options.fetch(:for, options[:model])
    @value_finder = FieldValueFinder.new(model, @view)

    yield self
  end

  def text_field(field_label, field_id, **options)
    options[:class] = css_class(options)

    field_value = @value_finder.find(options, field_id)
    field_html  = @view.text_field_tag(field_id, field_value, options)

    inline_labeled_field(field_html, field_label)
  end

  def select(field_label, field_id, collection, **options)
    options[:class] ||= css_class(options)

    value      = @value_finder.find(options, field_id)
    collection = @view.options_for_select(collection, value)
    field_html = @view.select_tag(field_id, collection, options)

    inline_labeled_field(field_html, field_label)
  end

  def submit(title, **options)
    options[:class] ||= @submit_class

    @view.submit_tag(title, options)
  end

  private

  def inline_labeled_field(field_html, field_label)
    return field_html if field_label.nil?

    @view.content_tag(:label) do
      "#{field_label}: #{field_html}".html_safe
    end
  end

  def css_class(options)
    [@input_class].push(options[:class]).compact.join(' ')
  end

  class FieldValueFinder
    def initialize(model, view)
      @model, @view = model, view
    end

    def find(options, field_id)
      options.delete(:value)   ||
      find_in_filter(field_id) ||
      find_in_request(field_id)
    end

    private

    def find_in_filter(field_id)
      @model.try :get_filter_param, field_id
    end

    def find_in_request(field_id)
      @view.params[field_id]
    end
  end
end
{% endhighlight %}

## Wrapup

Our class is now ready to grow, and is less of a monolithic monster. Although it is a bit longer,
the concepts are clearer. It is by no means perfect, but that's exactly the point: you should not
try to refactor everything right away, not until you really understand what's going on. Sometimes
early refactoring leads to the wrong abstraction.

There are things to watch out for: In the future, if we ever need to render a form input with a
disposition other than "inline", our first approach would be to create one more private method, for example,
**block_labeled_field**.

This is a great opportunity to refactor out that bit, moving the code to another class in order to handle the
layout concern. Another good reason to do that is you will likely need to refactor the existing
**inline\_labeled\_field** method to share functionality with the **block\_labeled\_field** method.
You will not want to bloat your **SearchForm** class with that, will you?

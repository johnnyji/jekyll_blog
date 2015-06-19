---
layout: post
title: "Ideal Validation Error Rendering in ERB!"
date: 2015-04-18
categories: Rails
---

So I've been playing around with ActiveRecord validation errors and I think I've found the ideal way I want to render them from now on.

When your validation fails you want to ideally render a new page to populate your current page with error messages. The most convenient way to to this would be.

{% highlight erb %}
<% if @model.errors.any? %>
  <% @model.errors.full_messages.each do |message| %>
    <%= message %>
  <% end %>
<% end %>
{% endhighlight %}
<br>

But the problem with this formatting is that it isn't exactly appealing to the eye. The better way to render errors would be to ideally display them each beside or above their respective form fields. In order to do so, all you would need to do is access the paticular error by passing the attribute as a key. Such as:

{% highlight erb %}
<%= @model.errors[:title] %>
<%# => ["Sorry! But that title is taken!"]
{% endhighlight %}
<br>

This will effectively return the custom error message you set in the form of an array, which will display as so when rendered in erb. So in order to only render the message itself, just simply select the last element of the array. So the entire line of code needed to render a specific error message without using a loop would be:

{% highlight erb %}
<%= @model.errors[:title].last %>
<%# => Sorry! But that title is taken!
{% endhighlight %}
<br>

Now you're free to render error messages WHEREVER you want, so go ahead and style the fuck outta that page :)


---
layout: post
title: "Scoping validations in ActiveRecord"
date: 2015-04-22
categories: Rails
---

In <em>ActiveRecord</em>, you can add something called `:scope` to your validations, which simply means validate an object WITHIN the scope of another object.

So if you had a `Upvote` class which needed to validate if a user has upvoted on a post already, the proper validation would be:

{% highlight ruby %}
class Upvote < ActiveRecord::Base
  belongs_to :post
  belongs_to :user

  validates :post_id, uniqueness: { true, scope: :user_id }

end
{% endhighlight %}
<br>

This will validate the uniqueness of the `post_id` WITHIN the scope of the `user_id`, as to make sure that there is never a duplicate `post_id` for a single `user_id`.

This will properly ensure that a post cannot be liked by a user more than once.

 
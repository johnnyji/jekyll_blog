---
layout: post
title: "STI stands for Single Table Inheritance"
date: 2015-04-29
categories: Rails
---

Today's topic that peaked my inheritance was <strong>Single Table Inheritance</strong>, which is having multiple models inherit their attributes from a base model. In Rails, STI occurs when all your models inherit from `ActiveRecord::Base`.

So in what situation would we best benefit from STI? Well when a model in your database as many `types` of itself. The best example I can think of is a User model. Imagine there are many "type" of users, such as:

<br>
<strong>Users</strong>
<ul>
  <li>Guests</li>
  <li>Admin</li>
  <li>Verified</li>
  <li>Banned</li>
</ul>
<br>

In a case such as this, STI is perfect because all of these sub-categories are their own objects with their own attributes and abilities, however they're all nested under the User model.

First you would need to add a `:type` column as a string to the `User` model in migrations, and Rails will automagically specify that column with the sub-category model name depending on what type of User was created or updated.

{% highlight ruby %}
class AddTypeToUsers < ActiveRecord::Migration
  def change
    add_column :users, :type, :string
  end
end
{% endhighlight %}
<br>

Now whenever you create the `Admin` or `Guest` or whatever model; you simply need to make those models inherit from `User` which is subsequently inherit from `ActiveRecord::Base`.

Just like that, you now have multiply types of users whom can also all have their own unique attributes and permissions!
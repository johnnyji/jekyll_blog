---
layout: post
title: "Creating Multiple Associations Between Two Models in ActiveRecord"
date: 2015-06-24
categories: ActiveRecord
---

Currently in one my project, I ran into an interesting association problem that I'd previously not dealt with before. Which is specifying different kinds of associations for the same model.

You're probably saying <em>"Wait whaaaa...?"</em>. Don't worry, I can relate because that's exactly what I said. So let's lay out an example scenario and see how we can approach solving it:

{% highlight ruby %}
class Production
  # we want to be able to find out the hiearchy of our cast on the production. So ideally we would have something like this:
  has_one :lead_actor
  has_many :supporting_actors
end

class Actor
  # we want to be able to find the different types of roles the actor has portrayed. So ideally we want:
  has_many :leading_performances
  has_many :supporting_performances
end

# Ideally if our production of "Lion King" had a lead actor "Jacob", when we query lead performances for "Jacob", we should see "Lion King". Same goes for the supporting actors as well.
{% endhighlight %}
<br>

So in this example, we have two simple models; `Actor` and `Production`. The only associations ever made is between the production and the actor, however a production can have many types of actors such as `leading_actor` and `supporting_actors`.

So why not just create an STI and have different types of actors?

{% highlight ruby %}
class LeadingActor < Actor; end
class SupportingActor < Actor; end
{% endhighlight %}
<br>

Well it's not quite that simple because an actor will not always be a leading actor. An actor can be a leading role on one production, but a supporting role on another. So in order to solve this issue, we must approach it from the association side of things. Here's what I came up with after about an hour of playing around with different options:

{% highlight ruby %}
# main models

class Production
  # creates two associations for the same model, specifying that model with a through relationship and :source
  has_one :leading_role
  has_many :supporting_roles
  has_one :leading_actor, through: :leading_role, source: :actor
  has_many :supporting_actors, through: :supporting_roles, source: :actor
end

class Actor
  # creates two associations for the same model, specifying that model with a through relationship and :source
  has_many :leading_roles
  has_many :supporting_roles
  has_many :leading_performances, through: :leading_roles, source: production
  has_many :supporting_performances, through: :supporting_roles, source: production
end

# through association models

class SupportingRoles
  belongs_to :movie
  belongs_to :actor
end

class LeadingRoles
  belongs_to :movie
  belongs_to :actor
end
{% endhighlight %}
<br>

Here our production has both lead and supporting actors due to a `through` relationship, we are also specifying through `source: :actor` that both associations are towards the `Actor` model.

`source` is an argument for an ActiveRecord association that specifies the model being associated to from the through model, if the name of the association is not the same as the name of the associated model. Read more about it [here][sourcepost].

Back to the topic at hand, we can now also query the lead and supporting performances of our actors because of the same `through` relationship that connects the two models, and we will know that `leading_performances` or `supporting_performances` will return the actual production because we once again specified that through the `source`.

And thats it! Now we have two models that can be associated with each other in multiple different ways and names; achieved by specifying the through models and the source model of the association.

[sourcepost]: http://stackoverflow.com/questions/4632408/need-help-to-understand-source-option-of-has-one-has-many-through-of-rails
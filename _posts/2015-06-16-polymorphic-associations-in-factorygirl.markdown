---
layout: post
title:  "Polymorphic Associations in FactoryGirl"
date:   2015-06-16 13:29:08
categories: RSpec
---
Before we begin. Please set `config.use_transactional_fixtures = true` in your spec helper. I found that when I wasn't doing this, Factory Girl was creating duplicate instances of my models and because of that, validations were failing.

So recently at work I've been refactoring the user models into polymorphic associations. Part of that process was introducing new factories for these polymorphic associations in Factory Girl. The associations are as follows:

{% highlight ruby %}
class Profile
  belongs_to :profileable, polymorphic: true
end

class Account
  belongs_to :accountable, polymorphic: true
end

class Student
  has_one :profile, as: :profileable
  has_one :account, as: :accountable
end

class Teacher
  has_one :profile, as: :profileable
  has_one :account, as: :accountable
end
{% endhighlight %}
<br>
 
Breaking down the models this way allows us to seperate the concerns of authentication, profile information and teacher/student specific information into their own datasets. I favour this over STI because here, there are no `nil` columns anywhere.

So the question remains, how do we create factories for such associations? Well first we need to create a profile and account factory and specify them as polymorphic:

{% highlight ruby %}
FactoryGirl.define do
  factory :account, class: Account do
    email                  { Faker::Internet.email }
    password               { "supersecret" }
    password_confirmation  { "supersecret" }
    association :accountable
  end
end

FactoryGirl.define do
  factory :account, class: Account do
    first_name             { Faker::Name.first_name }
    last_name              { Faker::Name.last_name }
    association :profileable
  end
end
{% endhighlight %}
<br>

We associate the factories to their polymorphicable (is that a word?), because in the model, they belong to their polymorphicable. Now it's time to create the student and teacher factories:

{% highlight ruby %}
FactoryGirl.define do
  factory :student, class: Student do
    trait :trial          { subscription_status: 0 } 
    trait :subscribed     { subscription_status: 1 } 

    after(:build) do |student|
      student.account = build(:account, accountable: student)
      student.profile = build(:profile, profileable: student)
    end

    after(:create) do |student|
      student.account.save!
      student.profile.save!
    end
  end
end
{% endhighlight %}
<br>

For the sake of scrolling, I won't create the teacher factory because it's the same idea. When creating the factories for the models that have the polymorphic model, you simply add on the polymorphic model in an `after(:build)` and `after(:create)` block and specify the polymorphicable to be the model the factory is for.

<b>TADA. Polymorphic associations in FactoryGirl... Done!

---
layout: post
title: "Lighthouse Labs W2D3: RSpec stubs and mocks"
date: 2015-04-08
categories: RSpec
---

RSpec stubs and mocks create instances of not yet existing objects on the fly and give them non-existent methods which return specific outputs that you declare.

Stubbing and mocking allow you to test other methods that depend on the method that you stubbed or the class that you mocked; either because you haven't created those methods/classes yet, or because running them would instrumentally slow down your test suite; or because you know the exact output of the methods and running them would be trivial or irrelevant. 

If you wanted to test the `rob_bank(bank)` method on an instance of the Robot class but you haven't created the Bank class yet, how would you test the robot's `rob_bank(bank)` methods?

{% highlight ruby %}
bank = instance_double("Bank", repository: 1000000, name: 'RBC')
allow(bank).to receive(:robbed).and_return(1000)

#Stubbing in a fake method which returns a value we set to our mocked bank object. This is what we would want our actual method on the actual bank object to return.

expect(@robot.rob(bank).to eq(1000)
{% endhighlight %}

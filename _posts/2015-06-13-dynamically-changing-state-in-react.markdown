---
layout: post
title:  "Dynamic state changing in React.js"
date:   2015-06-13
categories: React
---

One of the headaches I had with React was trying to figure out a general function that could update any state associated with the element that updated it. There were a few initial issues that I faced:

<br>
<blockquote>
  <li>How could I distinguish which state was associated with which element?</li>
  <li>How could one function change any state for any element?</li>
  <li>What if the state has nested attributes? Such as a `user` state</li>
</blockquote>
<br>

The following are two functions that I've spent some time coming up with. Both will dynamically change any state so long as the parameters are passed to them.

`changeStateAttributeValue` changes a single attributes on a given state, i.e. changing the email attribute on a state called user:

{% highlight javascript %}
changeStateAttributeValue: function(stateName, attributeKey, attributeValue) {
  var newState = this.state; 
  var stateBeingChanged = this.state[stateName];
  stateBeingChanged[attributeKey] = attributeValue;
  newState[stateName] = stateBeingChanged;
  this.setState(newState);
}
{% endhighlight %}
<br>

`changeState` simply changes a single state on a component given the state name and it's new value:

{% highlight javascript %}
changeState: function(stateName, value) {
  var newState = {};
  newState[stateName] = value;
  this.setState(newState);
}
{% endhighlight %}
<br>

I wish I would have had this code to begin with! This would have saved so much time! <b>DRY FTW.
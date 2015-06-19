---
layout: post
title: "Update Addon in React.js"
date: 2015-06-03
categories: React
---

React addons are freakin amazing. They essentially allow you to attach onto states when mutating them. Such that if you have a state which is an object with many attributes or a state that is an array that needs pushing onto. You can easily use the React update addon for the trick. Here's an example.

Assume that you have an array `[1, 2, 3, 4]` as a state on a React component. The regular way to handle adding elements onto this array would be to clone the array, make adjustments to the clone and reset the clone as the state like so:

{% highlight js %}
handleNewItems: function(items) {
  var newArray = this.state.array;
  newArray.push(items);
  this.setState({ array: newArray });
}
{% endhighlight %}
<br>

While this is fine for such a small task like adding extra items onto an array, it would get troublesome for larger tasks such as a state is an object that has other nested objects with many attributes, and maybe you only want to change on attribute on a nested object. This approach can start to get messy real fast. So here's where <strong>React update addon</strong> comes in. Which is facebook's built in way to solving the exact same problem pretty much, but just with cleaner code and optimized performance:

{% highlight js %}
handleNewItems: function(items) {
  var newArray = React.addons.update(this.state, { 
    array: { $push: items } 
  });
  this.setState(newArray);
}
{% endhighlight %}
<br>

Here we're using React's build in update function to directly update the entire state as passed in through the first argument, and the second argument is an object specifying which state we're changing and what we're doing. In this case we're `$push` to push on the new items. Afterwards we can simply reset the entire state.

This can also be done with multiple other functions such as `$set`, `$apply` etc... Checkout the [official docs][react-update] to learn more about Immutability helpers such as the update function in React.

[react-update]: https://facebook.github.io/react/docs/update.html
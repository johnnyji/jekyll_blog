---
layout: post
title: 'Binding React Component Functions in ES6'
date: 2015-08-13
categories: React
---

Transitioning React components to ES6 isn't all different and seemed to have flowed quite nicely, until I learnt that I had to manually bind all my component's functions to `this`. I felt like someone had just squirted ketchup all over my Oreo Sundae.

It turns out that one lovely thing about `React.createClass` was that it was automatically binding all the component's functions. So whats the ES6 solution for this?

We'll we could manually bind every function in our component like such:

{% highlight javascript %}
import React from 'react';

export default class LoginHandler extends React.Component {
  constructor(props) {
    // ...
    this.verifyEmail = this.verifyEmail.bind(this);
    this._verifyPassword = this.verifyPassword.bind(this);
    this._verifyPasswordConfirmation = this.verifyPasswordConfirmation.bind(this);
  }
  _verifyEmail(e) {
    //...
  }
  _verifyPassword(e) {
    //...
  }
  _verifyPasswordConfirmation(e) {
    //...
  }
}
{% endhighlight %}
<br>

<strong>That seriously sucks.</strong> Having to type all that for each function in every component makes my skin cringe. So instead, let's write our own class that every component can inherit from, to make this code alot more DRY and easier to the eye.

{% highlight javascript %}
import React from 'react';

export default class ReactTemplate extends React.Component {
  // (...params) is the new Rest Parameters in ES6 that allow multiple number of parameters
  _bindFunctions(...funcs) {
    funcs.forEach(func => this[func] = this[func].bind(this));
  }
}
{% endhighlight %}
<br>

Now that we've defined our `ReactTemplate`, we can extend from that on all our components. Later on, we can also add more helper functions to this template when necessary.

{% highlight javascript %}
import React from 'react';
import ReactTemplate from '...';

export default class LoginHandler extends ReactTemplate {
  constructor(props) {
    // ...
    // this.verifyEmail = this.verifyEmail.bind(this);
    // this._verifyPassword = this.verifyPassword.bind(this);
    // this._verifyPasswordConfirmation = this.verifyPasswordConfirmation.bind(this);
    this._bindFunctions('_verifyEmail', '_verifyPassword', '_verifyPasswordConfirmation');
  }
  _verifyEmail(e) {
    //...
  }
  _verifyPassword(e) {
    //...
  }
  _verifyPasswordConfirmation(e) {
    //...
  }
}
{% endhighlight %}
<br>

So much nicer. <strong>Ahhh.......</strong>
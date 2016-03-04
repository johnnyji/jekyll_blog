---
layout: post
title: 'Performance Tweeking in React.js using Immutable.js'
date: 2016-03-03
categories: React
---

Over the past little bit I've began to work more and more about speed and optimization in React. I've begun to notice that the more complex my apps got, the longer it would take them to render and re-render. Even React can't save us from eventual performance slowdown...

I watched a [great talk][talk] this morning given by my boss [Ev][ev], who is one of the smartest developers I know. In this talk he touches on things that tend to slow down our React applications, and it boils down to two main reasons:

1. Rendering to the DOM is insanely slow. Avoid it as much as you can.
2. Re-rendering to the DOM is just as painful. Avoid it as much as you can.

Okay, so clearly there's a direct message here; <b>avoid unnecessary writes to the DOM whenever possible</b>. Let's take a look at this in the context of React.js.

There is really no escaping the initial `render` process (otherwise users would not see anything on your page!). Knowing this, our best bet is to reduce the amount of re-renders to as little as we possibly can. This is the perfect job for `shouldComponentUpdate`.

As a part of React's [lifecycle][lifecycle], `shouldComponentUpdate` is a method that returns a boolean, which determines if the component needs to update or not.

{% highlight javascript %}
  // Re-render the component only if the message prop
  // will change on the next iteration.
  shouldComponentUpdate(nextProps, nextState) {
    return this.props.message !== nextProps.message;
  }
{% endhighlight %}

`shouldComponentUpdate` by itself already seems really useful for determining if a component should re-render. Simply put, if the props/state don't change, then don't re-render. Voila!

But wait... This is can get really ugly really fast. I can think of two general cases where your `shouldComponentUpdate` can get out of hand.

### 1. Too many props and state to check
{% highlight javascript %}
  shouldComponentUpdate(nextProps, nextState) {
    return (
      this.props.message !== nextProps.message ||
      this.props.firstName !== nextProps.firstName ||
      this.props.lastName !== nextProps.lastName ||
      this.props.avatar !== nextProps.avatar ||
      this.props.address !== nextProps.address ||
      this.state.componentReady !== nextState.componentReady
      // etc...
    );
  }
{% endhighlight %}

In this example, it's extremely easy to remove or add a prop/state to the component and forget to do the same in `shouldComponentUpdate`.

### 2. Deeply nested props/state

{% highlight javascript %}
  /*
    this.props.user = {
      name: {
        english: {
          first: 'Johnny',
          last: 'Ji'
        }
      },
      age: 21,
      height: // etc...
    };

    nextProps.user = {
      name: {
        english: {
          first: 'Johnny',
          last: 'Depp' // This attribute has changed!
        }
      },
      age: 21,
      height: // etc...
    };
   */

  shouldComponentUpdate(nextProps, nextState) {
    return this.props.user !== nextProps.user;
  }

  // Even though the two objects are different, `shouldComponentUpdate`
  // will return false.
{% endhighlight %}

In this example, we have less props but more deeply nested props (which is good). However, we can no longer use a simple `===` equator, we have to compare the two objects to see if they're different. What's even worse is that because both are nested, we have to recursively do a deep comparison.

<br /><br />
So why does this suck? Because deep comparisons are expensive to perform, and when done on every prop/state of every component over an entire app, could leave a nasty scar.

`shouldComponentUpdate` is actually so useful, React has an add-on helper mixin called `PureRenderMixin` that out-of-the-box implements it for you. All you need to do is decorate or mixin your component with the functionality. Unfortunately `PureRenderMixin` also only performs shallow comparisons; which still doesn't solve our problem.
<br /><br />

<h2>So what do I do? Answer: Immutable.js</h2>

Immutable.js is an amazing library that provides us with a way of having immutable datatypes in JavaScript.

{% highlight javascript %}
  Immutable.List([1,2,3])             // Like an array, but immutable
  Immutable.Map({a: 1, b: 2, c: 3})   // Like an object, but immutable
  // and so much more, etc...
{% endhighlight %}

Immutable.js not only allows us to bring functional paradigms such as immutability into JavaScript, but it also does a little thing

[ev]: https://github.com/globexdesigns
[talk]: https://www.youtube.com/watch?v=Jv18_gdAhGg
[lifecycle]: https://facebook.github.io/react/docs/component-specs.html
---
layout: post
title: 'Performance Tweaking in React.js using Immutable.js'
date: 2016-03-03
categories: React
---

Over the past little bit I've had troubles with optimization in React. I've begun to notice that the more complex my apps got, the longer it would take them to render and re-render. Even React can't save me from eventual performance slowdown (mostly due to my own stupidity)...

I watched a [great talk][talk] this morning given by my boss [Ev][ev], who is one without a doubt one of the smartest developers I know. In this talk he touches on things that tend to slow down our React applications, and it boils down to two main culprits:

<br />
<blockquote>
  <div>1. Rendering to the DOM is insanely slow. Avoid it as much as you can. </div>
  <div>2. Re-rendering to the DOM is just as slow. Avoid it as much as you can.</div>
</blockquote>
<br />

Okay, so clearly there's a direct message here; <b>avoid unnecessary writes to the DOM whenever possible</b>. Let's take a look at this in the context of React.js.
<br /><br />

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
<br /><br />

But wait... This is can get really ugly really fast. I can think of two general cases where your `shouldComponentUpdate` can get out of hand.

<h3><b>1. Too many props and state to check</b></h3>
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
<br /><br />

<h3><b>Deeply nested props/state</b></h3>
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

So why does this suck? Well because deep comparisons are expensive to perform, and when done on every prop/state of every component over an entire app, could leave a nasty scar.

`shouldComponentUpdate` is actually so useful, React has an add-on helper mixin called `PureRenderMixin` that out-of-the-box implements it for you. All you need to do is decorate or mixin your component with the functionality. Unfortunately `PureRenderMixin` also only performs shallow comparisons; which still doesn't solve our problem.
<br /><br /><br />

<h2>So what do I do? Answer: Immutable.js</h2>

[Immutable.js][immutable] is an amazing library that provides us with a way of having immutable datatypes in JavaScript.

{% highlight javascript %}
  Immutable.List([1,2,3]);             // Like an array, but immutable
  Immutable.Map({a: 1, b: 2, c: 3});   // Like an object, but immutable
  // and so much more, etc...
{% endhighlight %}

Immutable.js not only brings functional programming perks such as immutable datatypes into JavaScript, but it can also allows us to optimize the hell out of our React code. But first we need to understand how Immutable.js achieves immutability in a language at allows for mutations.

Immutable.js is just a collection of methods that will take your JavaScript datatypes as arguments and create an object that stores them. Under the hood, it actually works something like this:

{% highlight javascript %}
  const user = {
    firstName: 'Johnny',
    lastName: 'Ji'
  };

  const immutableUser = Immutable.Map({
    firstName: 'Johnny',
    lastName: 'Ji'
  });

  // immutableUser is now an instance of `Immutable.Map`, which is actually a JavaScript object containing information
  // about the `user` object we passed it. You can imagine it as something like this =>
  //
  // {id: 1, ref: 2bfe829d, size: 2, _actualData: {firstName: 'Johnny', lastName: 'Ji'}, methods: {...}};
{% endhighlight %}

Of course this is not actually how `Immutable.js` stores your data, but the structure and the logic is the same. It is nothing more an object wrapper that make sure you can't mutate the contained data, thus achieving the effect of <em>immutability</em>.
<br /><br /><br />

### What does this have to do with optimizing React?

Here it comes. Because `Immutable.js` records are immutable, everytime a change is made, a new copy of the record is returned, and the previous one is never altered. But the best part about that is that every new record also comes with a new `reference` attribute.

{% highlight javascript %}
  const car = Immutable.fromJS({
    exterior: {
      details: {
        color: 'blue'
      }
    }
  });

  const differentCar = car.setIn(['exterior', 'details', 'color'], 'red');

  // The original `car` was never changed, instead a new copy of `car` with the color attribute set to red
  // was created and set to variable called `differentCar`. Most importantly, notice that the reference of the
  // two maps have changed:
  //
  // `car` =>           {size: ..., id: ..., _actualData: ..., ref: 123}
  // `differentCar` =>  {size: ..., id: ..., _actualData: ..., ref: 738}
{% endhighlight %}

If all changes to `Immutable.js` records return a new record with a new `reference` id, that effectively means not matter how deeply nested the change, we will still be aware that the new record has INFACT changed. <b>We can deep compare Immutable.js datatypes by simply checking if the references are different</b>.

I'll say it again: <b>We can deep compare Immutable.js datatypes by simply checking if the references are different.</b>

Didn't our whole problem with `shouldComponentUpdate` and `PureRenderMixin` have to do with the inpracticality of performing deep comparisons on our props/state?
<br /><br /><br />

### PureRenderMixin + Immutable.js === Problem Solved!

Because `Immutable.js` creates new records with new references upon any change to the data, we can do deep comparisons of current props/state to next props/state for next to nothing! In fact, so long as you use `Immutable.js` for your datatypes, `PureRenderMixin` will work and "deep compare" out-of-the-box without you having to do any extra configurations!

The ability to use `PureRenderMixin` on any component in our app regardless of the props/state structure complexity is amazing. This allows us to make certain that components throughout our app are only ever re-rendering when they truely need to.

<br/><br/>
<b>Less re-rendering === Boost in performance.</b>

[immutable]: https://facebook.github.io/immutable-js/
[ev]: https://github.com/globexdesigns
[talk]: https://www.youtube.com/watch?v=Jv18_gdAhGg
[lifecycle]: https://facebook.github.io/react/docs/component-specs.html
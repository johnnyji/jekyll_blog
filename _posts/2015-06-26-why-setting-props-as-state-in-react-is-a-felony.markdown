---
layout: post
title: "Why Setting Props as State in React.js is Blasphemy"
date: 2015-6-26
categories: React
---

Okay, maybe blasphemy is a bit of a harsh word; but seriously, avoid setting `state` from `props` at all cost if you can in React components!

When I started out with React, I made this mistake countless times, and like the noob that I was <strike>and still probably am</strike>, I decided to I to set state as props that were passed through to the component and declared myself a "genius" for doing so... Oh how I wish I could go back and slap myself in the face.

Let's take a step back and define what `props` and `states` are:

<strong>Props:</strong>
<blockquote>
  Props are short for properties. These are static properties on a React component that are immutable (cannot be changed).
</blockquote>

<strong>State:</strong>
<blockquote>
  State is mutable, and defines at any given time, the current state of the React component that is being rendered.
</blockquote>
<br>

Thats the key right there, <strong>Immutable vs. Mutable</strong>. Because `state` if mutable, it is passed down to child components as `props` which are immutable. 

This ensures that the only way for the child component to render something different or change is if the state of the parent component has changed; causing a rerender of all the child components whom have inherited it's state through props. This is the central idea of React's <strong>unidirectional data flow</strong>.

Here are two examples:

{% highlight javascript %}
  // BAD EXAMPLE
  class ChildComponent extends React.Component {
    constructor(props) {
      this.state = {
        age: props.age
      }
    }
    render() {
      return (
        <div>
          <p>`I am ${this.state.age} old`</p>
        </div>
      );
    }
  }

  // GOOD EXAMPLE
  class ChildComponent extends React.Component {
    constructor(props) {
      // no need to set state here because the age is passed down from the parent component through props
    }
    render() {
      return (
        <div>
          <p>`I am ${this.props.age} old`</p>
        </div>
      );
    }
  }

  ChildComponent.propTypes = {
    age: React.propTypes.number.isRequired
  }
{% endhighlight %}
<br>

The difference between the first and the second example is that when the `this.state.age` on the parent of the `ChildComponent` changes, the second example will rerender. This is because the new age prop is being used in the render function, causing the `ChildComponent` to rerender.

However, the first example will NOT rerender unless the component is unmounted and remounted again; because `state` is only ever set when the component initially mounts.

It was groundbreaking for me when I finally realized why I shouldn't be setting `state` from `props`, and I hoped this helped you come to a similar realization and <em>dancing around in your room</em> feeling.
<br><br>

<h3><strong>Oh yeah!</strong></h3>
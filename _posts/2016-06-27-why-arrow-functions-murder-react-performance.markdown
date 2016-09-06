---
layout: post
title: 'Why Anon Functions Murder Performance in React'
date: 2016-06-27
categories: React
---

If you've ever experienced performance issues whilst using React.js, it's likely because your React components are needlessly re-rendering. Luckily, React provides an easy out of the box solution which is `shouldComponentUpdate`. If you've never seen or used `shouldComponentUpdate`, I made a post about it [here][perfPost].

`shouldComponentUpdate` <b>shallow compares</b> the props/state of the component and only when either has changed, does the component re-render; thus cutting out needless re-renders.
<br/>
<br/>

<h3>So... What's the problem?</h3>

Well the problem is, there's been a pattern that I've been seeing quite a lot in the React.js community; using anonymous (arrow) functions as callback props. Here are two examples, the first with arrow functions, and the latter without. It's immediately clear why people prefer the arrow function approach:
<br/>
<br/>

<b>*with* ES6 Arrow Functions:</b>
{% highlight javascript %}
  // ...

  render() {
    return (
      <div>
        <CustomButtonComponent
          label='Click me to increment!'
          onClicked={() => { this.setState({count: this.state.count + 1}) }} />
        <CustomButtonComponent
          label='Click me to decrement!'
          onClicked={() => { this.setState({count: this.state.count - 1}) }} />
      </div>
    );
  }
  
  // ...
}
{% endhighlight %}

<b>*without* ES6 Arrow Functions:</b>
{% highlight javascript %}
  // ...

  render() {
    return (
      <div>
        <CustomButtonComponent
          label='Click me to increment!'
          onClicked={this._handleIncrementView} />
        <CustomButtonComponent
          label='Click me to decrement!'
          onClicked={this._handleDecrementView} />
      </div>
    );
  }

  _handleIncrementCount = () => {
    this.setState({count: this.state.count + 1});
  };
  
  _handleDecrementCount = () => {
    this.setState({count: this.state.count - 1});
  };

  // ...
}
{% endhighlight %}
<br/>

Looking at this code, I would probably pick the first one. With the introduction of arrow functions in ES6, it has never been easier to write anonymous functions, the first example using arrow functions looks very clean and legible (if I do say so myself :bowtie:).

<b>HOWEVER</b>, there is always more than meets the eye. The first example is actually REALLY BAD. The first component will cause the `CustomButtonComponent` to re-render needlessly, even when we've optimized the component with `shouldComponentUpdate`.
<br/>
<br/>

<h3>So why are arrow functions bad exactly?</h3>

Arrow functions used in this context are what's known as `Anonymous Functions` in JavaScript. Anonymous functions, just as they their name hints at, have no identity. They are functions that are created, used and discarded just as quickly.

We use anonymous functions all the time is JavaScript, mostly as callback functions to execute some quick functionality. Therefore, the logical thought is to use anonymous functions for our React component callback props as well. However, this will be the <em><b>silent killer</b></em> of your React app's performance. Here's why:

{% highlight javascript %}
class Main extends React.Component {
  // ...
  render() {
    // We should NEVER pass anonymous functions as props to React components unless
    // we absolutely need to. Anonymous functions are redefined on every render cycle, which means
    // the function you pass to your child component is different every time.
    //
    // This will break shallow comparing on your child component, as the prop will be different every time.
    return (
      <div>
        <p>This button has been clicked {count} times</p>
        <CustomButtonComponent
          label='Click me!'
          onClicked={() => // ... } />
      </div>
    );
  }
}


// ... In another file ...

class CustomButtonComponent {
  shouldComponentUpdate(nextProps) {
    // This will ensure that our component only ever re-renders
    // if it's props have changed
    return (
      this.props.onClicked !== nextProps.onClicked ||
      this.props.label !== nextProps.label
    );
  }

  render() {
    return <button onClick={this.props.onClicked}>{label}</button>;
  }
}
{% endhighlight %}

An anonymous function by nature has no reference, thus they will be redefined/recreated on every component render cycle. In our case, that means everytime `Main` re-renders, it redefines the `onClicked` prop of the `CustomButtonComponent`, forcing `CustomButtonComponent` to re-render even if has been optimized with `shouldComponentUpdate`. <b>This breaks the intended behaviour of `shouldComponentUpdate`</b>.
<br/>
<br/>

<b>Here's how we should rewrite this file so `CustomButtonComponent` will only render as needed:</b>
{% highlight javascript %}
class Main extends React.Component {
  // ...
  render() {
    // Because we've named the callback prop `this._handleClicked`, we're now passing a FUNCTION REFERENCE and
    // no longer creating a new function on every render cycle. The function reference will not change throughout
    // the lifecycle of `Main`, therefore it will never cause `CustomButtonComponent` to re-render.
    return (
      <div>
        <p>This button has been clicked {count} times</p>
        <CustomButtonComponent onClicked={this._handleClicked} />
      </div>
    );
  }

  _handleClicked = () => {
    // ...
  };
}
{% endhighlight %}
<br/>

Here we're passing the reference of `this._handleClicked`, and therefore not creating a new function on every render. Now `CustomButtonComponent` will receive the same callback prop everytime and won't needlessly re-render. But don't just take my word for it, I've created a working [jsfiddle][fiddle] to demostrate the downside of using arrow functions as callback props, see for yourself [here][fiddle].

The downside to this approach is that you have to write some more code (most of which will boil a lot of plates); however the flip side is that you end up with optimized performance, and I think that's a pretty big win.
<br/>
<br/>


[fiddle]: https://jsfiddle.net/johnnyji/mtkjc5on/
[lifecycle]: https://facebook.github.io/react/docs/component-specs.html
[perfPost]: http://johnnyji.me/react/2016/03/03/performance-optimization-in-reactjs-using-immutablejs.html

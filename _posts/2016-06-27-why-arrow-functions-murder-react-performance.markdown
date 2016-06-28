---
layout: post
title: 'Why Arrow Functions Murder React.js Performance'
date: 2016-06-27
categories: React
---

If you've ever experienced performance issues whilst using React.js, it's likely because your React components are needlessly re-rendering.

Luckily, React provides an easy out of the box solution which is `shouldComponentUpdate`. If you've never seen or used `shouldComponentUpdate`, I made a post about it [here][perfPost].

<br />

`shouldComponentUpdate` <b>shallow compares</b> the props/state of the component and only when either has changed, does the component re-render; thus cutting out needless re-renders.

<h3>So... What's the problem?</h3>

Well the problem is, there's been a pattern that I've been seeing quite a lot in the React.js community; using anonymous (arrow) functions as callback props. Here are two examples, the first with arrow functions, and the latter without. It's immediately clear why people prefer the arrow function approach:
<br/>

<b>WITH ES6 arrow functions</b>
{% highlight javascript %}
class Main extends React.Component {
  
  state = {
    count: 0
  };

  render() {
    return (
      <div>
        <CustomButtonComponent
          label='Click me to increment!'
          onClicked={() => this.setState({count: this.state.count + 1})} />
        <CustomButtonComponent
          label='Click me to decrement!'
          onClicked={() => this.setState({count: this.state.count - 1})} />
      </div>
    );
  }
}
{% endhighlight %}
<br />

<b>WITHOUT ES6 arrow functions</b>
{% highlight javascript %}
class Main extends React.Component {
  
  state = {
    count: 0
  };

  render() {
    return (
      <div>
        <CustomButtonComponent
          label='Click me to increment!'
          onClicked={this._handleIncrementView} />
        <CustomButtonComponent
          label='Click me to decrement!'
          onClicked={() => this.setState({count: this.state.count - 1})} />
      </div>
    );
  }

  _handleIncrementCount = () => {
    this.setState({count: this.state.count + 1});
  };
  
  _handleDecrementCount = () => {
    this.setState({count: this.state.count - 1});
  };
}
{% endhighlight %}

Looking at this code, I would probably pick the first one. With the introduction of arrow functions in ES6, it has never been easier to write anonymous functions, this code looks very clean and legible (if I do say so myself :bowtie:).

<b>HOWEVER</b>, there is always more than meets the eye. The first example is actually REALLY BAD. The first component will cause the `CustomButtonComponent` to re-render needlessly, even when we've optimized the component with `shouldComponentUpdate`.

<h3>So why are arrow functions bad exactly?</h3>

[lifecycle]: https://facebook.github.io/react/docs/component-specs.html
[perfPost]: http://johnnyji.me/react/2016/03/03/performance-optimization-in-reactjs-using-immutablejs.html

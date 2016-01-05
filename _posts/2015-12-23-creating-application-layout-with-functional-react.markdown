---
layout: post
title: 'Creating Application Layout and UI with Functional React'
date: 2015-12-23
categories: React
---

I can't believe it's been almost 4 month since I've made a post. These past <strike>4</strike> 3 and a half month of my life has been insanely crazy for me. I have so many updates, so much to share! Here's a brief breakdown, hopefully I'll get around to dedicating posts for all of these:

  - I work at a [new place][cumul8]!
  - I've been diving hard into `redux`!
  - I've began writing a book!
  - I've began to build a new product which I'm very excited to release more info about soon!
  - I'm beginning to play with `elixir`!
  - I've been occupied with Star Wars taking over my existance.

But enough about that! I've learnt quite a bit about React over the last few months and I'd like to show you some things I've been doing - starting with changing the way I create application layout. Let's take a look at a simple app:

{% highlight javascript %}
class Dashboard extends React.Component {
  render() {
    return (
      <div className='dashboard-wrapper'>
        <ul className='post-list'>
          <li className='post-item'>{/*...*/}</li>
          <li className='post-item'>{/*...*/}</li>
          <li className='post-item'>{/*...*/}</li>
        </ul>
      </div>
    );
  }
}
{% endhighlight %}

Here we have a simple dashboard that shows a list of posts. But if we examine closer, that's not really what it is. It's actually a list of items inside a container, which for sure has other use cases than just posts.

Most likely throughout our app, we'll have commonly themed UI elements or even the exact same UI elements in more than one place. This sounds like a perfect situation to make use of React components!

Let's refactor the code above a bit more clear:

{% highlight javascript %}
// A list item within a list
class ListItem extends React.Component {
  render() {
    return (
      <li className={this.props.className + ' list-item'}>
        {this.props.children}
      </li>
    );
  }
}

// Any time we need a list in our app
class List extends React.Component {
  render() {
    return (
      <ul className={this.props.className + ' list'}>
        {this.props.children}
      </ul>
    );
  }
}

// Any time we need to wrap content in our app, this is great for setting min and max content
// content width.
class ContentWrapper extends React.Component {
  render() {
    return (
      <div className={this.props.className + ' content-wrapper'}>
        {this.props.children}
      </div>
    );
  }
}

// Now finally, here's our new `Dashboard` component
class Dashboard extends React.Component {
  render() {
    return (
      <ContentWrapper className='dashboard-wrapper'>
        <List className='post-list'>
          <ListItem className='post-item'>{/*...*/}</ListItem>
          <ListItem className='post-item'>{/*...*/}</ListItem>
          <ListItem className='post-item'>{/*...*/}</ListItem>
        </List>
      </ContentWrapper>
    );
  }
}
{% endhighlight %}

That `Dashboard` component is so much more <b>legible</b> than our previous one! Seperating out application layout and UI into React components allow us to:

 - Create self contained UI elements
 - Have a better visual hiearchy of our app
 - Reuse similar UI elements across our app faster

Once I started migrating all the <em>dumb</em> layout and UI components into their own components, I was so much more efficient at visualizing my app without even opening a web browser, because I knew what all of those components looked like, and I could easily compose them together to create a view.

<b>But wait</b> - you've probably noticed that one of the downsides of this is very repetitive and mundane code. Even worse, we're inherting all of `React.Component`'s functionalities and methods, when really all we're doing is rendering the child props, this can get a bit expensive as you start to build a lot of these layout components.
<br><br>

<b>FUNCTIONAL REACT TO THE RESCUE!</b>

The release of `React v0.14` introduced something very useful - <b>Functional React Components</b>, which look something like this:

{% highlight javascript %}
const List = (props) => {
  return (
    <li className={props.className + ' list'}>
      {props.children}
    </li>
  ) 
};

// Or to even shorten this...
const List = (props) => <li className={props.className + ' list'}>{props.children}</li>;

ReactDOM.render(
  <List className='post-list'>{/*...*/}</List>
);
{% endhighlight %}

Essentially, any component that only receives and renders props ("dumb" components), can now be written as pure functions, and rendered just like any other React component. The advantages to using functional components when possible are:
  
  - Faster to render because we don't need to inherit all the additional functionalities of `React.Component`
  - No need to declare components as `class` (functions are much more lightweight!)
  - Much less code to write

This is absolutely the perfect tool for our layout components. After a bit of refactoring using functional components, our application layout should now look like this:

{% highlight javascript %}

// Newly functional components
const ListItem = (props) => (
  <li className={props.className + ' list-item'}>
    {props.children}
  </li>
);

class List = (props) => (
  <ul className={props.className + ' list'}>
    {props.children}
  </ul>
);

const ContentWrapper = (props) => (
  <div className={props.className + ' content-wrapper'}>
    {props.children}
  </div>
);

// Now finally, here's our new `Dashboard` component
class Dashboard extends React.Component {
  render() {
    return (
      <ContentWrapper className='dashboard-wrapper'>
        <List className='post-list'>
          <ListItem className='post-item'>{/*...*/}</ListItem>
          <ListItem className='post-item'>{/*...*/}</ListItem>
          <ListItem className='post-item'>{/*...*/}</ListItem>
        </List>
      </ContentWrapper>
    );
  }
}
{% endhighlight %}
<br>

Fuck me, that's beautiful.

[cumul8]: http://www.cumul8.com
[reactrouter]: https://rackt.github.io/react-router/
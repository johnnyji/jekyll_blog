---
layout: post
title: "React State Management Through Reflux.js"
date: 2015-06-21
categories: React
---

React is an amazing library, no doubt about it. But every library has it's own quirks that make us all go <em>"If only that one thing was done better..."</em>.

For me, that quirk is having to invoke a callback function from a child component in order to change the state of it's parent component. Now this doesn't seem so troublesome when it's just the two layers deep, but imagine having to pass down props through numerous child components and having a chain of callbacks being fired so the parent component can receive the callback and update the state accordingly. Now imagine having to keep track of multiple props that fire multiple callbacks across multiple components. That to me, sounds like the definition of hell.

So enter <strong>Reflux</strong>. Reflux is a framework for an architechtural pattern called Flux. I won't get too much into Flux on this post, check out the [official docs][flux] to learn more.

Basically Reflux helps us keep track of our application state so that instead of passing around numerous callback functions, we can instead trigger events and have certain React components who need the data of those events listen for the triggers; bypassing the middle man completely! 

<em>(The middle in this case being all the other components that the callbacks would normally have to flow through to reach it's destination)</em>
<br><br>

Here's a small and not so bad example of the callback hell I was mentioning:

{% highlight javascript %}
// For the purposes of this demonstration I will be using ES5

var Parent = React.createClass({
  getInitialState: function() {
    return {
      name: "Johnny",
      sex: "male"
    }
  },
  handleChangeName: function(e) {
    this.setState({ name: e.target.value });
  },
  render: function() {
    return (
      <DisplayInfo
        name={this.state.name}
        sex={this.state.sex}
        handleChangeName={this.handleChangeName}
      />
    );
  }
});

var DisplayInfo = React.createClass({
  render: function() {
    return (
      <div>
        <h1>Name: {this.props.firstChildOnlyInfo}</h1>
        <h3>Sex: {this.props.sex}</h3>

        <InputField handleChangeName={this.props.handleChangeName} />
      </div>
    );
  }
});

var InputField = React.createClass({
  render: function() {
    return <input onChange={this.props.handleChangeName}></input>
    // this.props.handleNameChange will invoke the prop that it belongs to on it's parent component, which will in turn do the same to it's parent component until we've reached the very top level parent component, in which a function is invoked to change a state. The changed state is then passed back down as props through rerendering in order to alter each component accordingly.
  }
});

{% endhighlight %}
<br>

So here we can already see that even with only two levels of child component, we're having to tediously pass props (`handleChangeName`) around as functions so a callback chain can reach the parent component to change the state of `name`. Imagine doing this on a production scale application. Not ideal at all.
<br><br>

Here's how we would approach the same situation in <strong>Reflux</strong>:

{% highlight javascript %}
// Here's our new Reflux actions and stores!

var ParentAction = Reflux.createActions([
  // actions are simply an array of strings that direct to functions in stores.
  "changeName"
]);

var ParentStore = Reflux.createStore({
  init: function() {
    // we keep a track of the state on the store
    this.state: {
      name: "Johnny",
      sex: "male"
    };
    // this is telling our store to listen to any invoked actions in the ParentActions and invoke the corresponding function. The naming convention is to prepend the function in the store with the word "on". (i.e. if the action is "changeName", the corresponding store function is "onChangeName")
    this.listenTo(ParentActions);
  },
  onChangeName: function(newValue) {
    this.state.name = newValue;

    // trigger is a method in Reflux that triggers an event, allowing any listeners to react to the event. In this case, our Parent component in React is listening to this event.
    this.trigger({ name: this.state.name });
    //=> this will trigger: { name: "some value" }, and we just have to simply call this.setState(); on that in our React component.
  }
});


// Here's our React code!

var Parent = React.createClass({
  getInitialState: function() {
    return {
      name: "Johnny",
      sex: "male"
    }
  },
  componentDidMount: function() {
    // here we are listening to the ParentStore for any changes it triggers, and passing in onChange as the callback function that will handle the changes and update any state accordingly.
    this.unsubscribe = ParentStore.listen(this.onChange);
  },
  componentWillUnmount: function() {
    this.unsubscribe();
  },
  onChange: function(data) {
    // here the data is the new state being passed back, because the stores only have one trigger action, we're using lodash's _.pick to select only the key-value pairs that exists as states on this component. So if any other component's events are triggers, we're not adding any additional state onto this component by accident.
    var newState = _.pick(data, _.keysIn(this.state));
    this.setState(newState);
  },
  render: function() {
    return (
      <DisplayInfo
        name={this.state.name}
        sex={this.state.sex}
        // there's no longer need for callback props to be passed
      />
    );
  }
});

var DisplayInfo = React.createClass({
  render: function() {
    return (
      <div>
        <h1>Name: {this.props.firstChildOnlyInfo}</h1>
        <h3>Sex: {this.props.sex}</h3>
      
        <InputField />
      </div>
    );
  }
});

var InputField = React.createClass({
  handleChange: function(e) {
    // here we're telling the ParentStore to update the state of name
    ParentStore.changeName(e.target.value);
  }
  render: function() {
    return <input onChange={this.handleChange}></input>
  }
});
{% endhighlight %}
<br>

So as you can see here, we are no longer having to pass props around to invoke callbacks in order to change parent state.

<strong>TL;DR</strong> Anytime we have to change a state on a distant parent component, we simply call an action which will trigger a store function. The store will do it's calculation and trigger it's own event, which React components can listen to, and those components that are listening will automatically receive the new data, without us having to manually pass the data through callback props.

Reflux is an amazing tool and I've only begun to take a look at it, so I'm sure that I'll post more about it in the future! Until then, have fun Reacting!

[flux]: https://facebook.github.io/flux/docs/overview.html
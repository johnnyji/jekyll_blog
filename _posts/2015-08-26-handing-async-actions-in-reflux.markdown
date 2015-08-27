---
layout: post
title: 'Handling Async Actions in Reflux.js'
date: 2015-08-26
categories: CSS
---

I've been using Reflux for the past while now and up until recently, whenever I needed to make an API call, the workflow has been my React component triggering the action, the action emitting to the store and the store making an AJAX call and storing the result. Simple right?

It turns out that this isn't the best pratice for a couple of reasons:

<blockquote>
  <ol>
    <li>Stores are simply means of storing data. The dumber they are, the better</li>
    <li>The action of asking for data should be be moved into it's own utility to better seperate concerns</li>
    <li>If the AJAX call doesn't return the response your store expects, you can choose to not even acknowledge it in the store.</li>
  </ol>
</blockquote>
<br>

Keeping this in mind, let's redesign the way we make AJAX calls in Reflux.js!
<br>

Here we have a simple React component that triggers a `PostActions.loadPosts` before it mounts.

{% highlight javascript %}
export default class PostsList extends React.Component {
  componentWillMount() {
    PostActions.loadPosts();
  }
  componentDidMount() {
    this._unsubscribe = PostStore.listen(this._updateState);
  }
  componentWillUnmount() {
    this._unsubscribe();
  }
  _updateState(newState) {
    this.setState(
      // Updates the state of the component received from the store.
    );
  }
  render() {
    // ...
  }
}
{% endhighlight %}
 
 And here's our actions that handle that `loadPosts`

{% highlight javascript %}
var PostActions = Reflux.createActions({
  'loadPosts': { children: ['completed', 'failed'] },
  'someSyncAction': {},
  'youGetThePoint': {}
});
{% endhighlight %}

So basically what's happening here is when we make our `createActions` an object, we can assign a key to it called `children`, this has an array containing either `completed`, `failed` and also `progressed` (but we won't use `progressed` in this example).

`completed` and `failed` are just mirrors of resolving and rejecting a promise in JavaScript. Once our aysnc action completes, we can either trigger `completed` or `failed` based on the outcome of the action. Here's how.

{% highlight javascript %}
var PostActions = Reflux.createActions({
  'loadPosts': { children: ['completed', 'failed'] }
});

PostActions.loadPosts.listen(function() {
  // We will call our API for posts and return a promise, and based on the outcome of that promise, we either complete or fail the action.

  ApiCaller.sendAjaxRequest({
    url: '/api/posts',
    method: 'GET'
  })
  .then(this.completed)
  .catch(this.failed);

  // Your AJAX calls should be moved into it's own seperate API Util class, which I'll demonstrate later.
});
{% endhighlight %}

The `completed` and `failed` actions are then reflected in the store by attaching either suffixes onto the corresponding function's name in the store.

{% highlight javascript %}
var PostStore = Reflux.createStore({
  init: function() {
    this.state = { //... };
    this.listenToMany(PostActions);
  },
  onLoadPostsCompleted: function(result) {
    // when PostActions.loadPosts.completed() is called, we hit this function.
    // ... store the posts in state ... trigger state ...
    this.state.posts = result.data.posts;
    this.trigger(this.state);
  },
  onLoadPostsFailed: function() {
    // when PostActions.loadPosts.failed() is called, we hit this function.
    // ... trigger some error message ...
  }
});
{% endhighlight %}
<br>

Now that we understand the general workflow, let's hook up our API Util classes!

We need to create a class that acts as a wrapper for all our API calls. It may be very tempting to just fall back to using jQuery's `$.ajax` (which is awesome, I'm not knocking it one bit), but if that's all we're using jQuery for, then there's no need to include the entire dependency in our project. It'd be much easier to write our own AJAX wrapper, which would look something like this:

{% highlight javascript %}
export default class ApiCaller {

  // this method assumes that you are sending/receiving JSON
  // options: { url: ..., method:..., data:... }
  static sendAjaxRequest(options) {
    return new Promise((resolve, reject) => {
      let request = new XMLHttpRequest();
      request.open(options.method, options.url);

      request.onload = () => {
        let result = {
          status: request.status,
          data: JSON.parse(request.responseText)
        };
        if (result.status >= 200 && result.status <= 299) {
          console.log('Response: ', result.data);
          resolve(result);
        } else {
          console.log('Response: ', result.data);
          reject(result);
        }
      };

      request.onerror = () => {
        console.log('Response: ', result.data);
        reject({ status: 500, data: 'Connection error' });
      }

      console.log(`Sending ${options.method} Request to ${options.url}`);
      request.send(options.data);
    });
  }

}
{% endhighlight %}

Now that we have our API Util, let's hook it up to our actions.

{% highlight javascript %}
var PostActions = Reflux.createActions({
  'loadPosts': { children: ['completed', 'failed'] }
});

PostActions.loadPosts.listen(function() {
  // calling either this.completed or this.failed based on the result of our AJAX call

  Api.sendAjaxRequest({
    url: '/api/posts',
    method: 'GET'
  })
  .then(this.completed)
  .catch(this.failed);
});
{% endhighlight %}

And we just handle the outcome in our store with `onLoadPostsCompleted` and `onLoadPostsFailed` and we should be good!

That's the basic idea behind the workflow. The component will trigger an action, the action will call an API Util call that makes the API call and returns a promise, the action listens to that promise and calls either `completed` or `failed` based on the outcome of the promise, the store then has two seperate functions that respond to both the completed and failed events emitted by the action.
<br>

<strong>Happy AJAXing!</strong>
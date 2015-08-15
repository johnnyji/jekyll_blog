---
layout: post
title: 'Intro to React Router'
date: 2015-08-13
categories: React
---

Today has been a pretty awesome day for me. I had the chance to fully dive into setting up a React app in NPM (which will be another blog post on it's own). As I was setting up the app, I also had the chance to explore and implement [React Router][reactrouter].

I found React Router's nested routing architecture very intuitive and true to React's trickle down nature. Here's the step-by-step intro to how I found myself to implement the React Router. For a more comprehensive guide, I highly recommend hitting the [docs][reactrouter].

I'll assume that you have a React skeleton set up. Go ahead and run `npm i -S react-router`.

In our `app.js` (or whatever your entry point js file is), instead of rendering the React component using `React.render`, we're going to render the component using the router.

{% highlight javascript %}
//  app/shared/routes.js

import React from 'react';
import { Route } from 'react-router';

// Import any routes handler components that will be used
import AppHandler from '../components/app/appHandler';
import LoginHandler from '../components/app/loginHandler';

let routes = (
  <Route name='app' path='/' handler={AppHandler}>
    <Route name='login' path='/login' handler={LoginHandler} />
    <Route name='password-reset' path='/password_reset' handler={PasswordResetHandler} />
  </Route>
);

export default routes;
{% endhighlight %}

{% highlight javascript %}
// app/app.js

import React from 'react';
import Router from 'react-router';
import routes from './shared/routes'

Router.run(routes, Router.HistoryLocation, (Handler) => {
  React.render(<Handler />, document.body);
});
{% endhighlight %}
<br>

Our `app.js` file is calling `Router.run` which takes all our routes (imported from the `routes.js` file), and dependant on which route handler is called, renders that paticular component.

You'll see that the second argument the `Router.run` function takes is `Router.HistoryLocation`. By default, every route will be prefixed with a `#`. when you click a link that takes you to the "login" link, the URL path will actually be `http://localhost:8080/#/login`. In order to bypass this `#` key, we specify that we want to use `Router.HistoryLocation` to create our URLs. The resulting route will be `http://localhost:8080/login`.

In our `routes` file, we have specified that our most top level route is `/`, which points to the `AppHandler`. This is the setup for our `AppHandler`:

{% highlight javascript %}
// app/components/app/appHandler.js

import React from 'react';
import Router from 'react-router';
import { Link, RouteHandler } from 'react-router';


export default class AppHandler extends React.Component {
  render() {
    return (
      <div className='app'>
        <Link to='login'>Login</Link>

        // VERY IMPORTANT, this is what handles all our routes
        <RouteHandler />  
      </div>
    );
  }
}
{% endhighlight %}
<br>

We specify must `<RouteHandler />` at the very bottom of our component render in order for the router links to work. Now, whenever the user clicks on a `Link` component that we've imported from the router, the corresponding `<Route />` component in our `routes.js` will be called.

{% highlight javascript %}
// The link component's "to" prop will find the corresponding route component with the same "name" prop. These two props are the identifying factors that link the two components.

// this link...
<Link to='some-path'>Go to path!</Link>

// will hit this route!
<Route name='some-path' path='/some_url_path' handler={SomeReactComponent} />
{% endhighlight %}
<br>

React Router seems really amazing and I can wait to dig way deeper into it. This is just the tip of the router iceberg for me and I'll document my progress as I discover more and more amazing things about it!

[reactrouter]: https://rackt.github.io/react-router/
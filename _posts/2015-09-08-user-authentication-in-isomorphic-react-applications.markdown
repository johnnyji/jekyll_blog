---
layout: post
title: 'User Authentication In Isomorphic React Applications'
date: 2015-09-08
categories: React
---

I love apps that remember who I am and redirects me to certain pages on load, depending on if I'm logged in or not. I read an amazing post about [React and Flux Auth Flow][reactauthflow]. It does an exceptional job at walking through the process of user authentication using React, Flux and React Router...

However, this approach will only work with a pure client side application. I looked tirelessly online for a simple way to authenticate an isomporhic application, but there wasn't much on subject. 

Through playing around, I've found my own way to implement a very similar auth flow that works amazing well with server side rendering! The main difference between my implementation and the other ones is that I've opted out of using `willTransitionTo`, which I'll explain why at the end.

<em>(For brevity, I will keep my code snippets fairly simple, this is by no means an example of production grade code).</em>
<br><br>

<strong>Here's what we'll be using:</strong>

- `React` duh
- `React Router` for our routing purposes
- `Reflux` as the flux implementation
- `Node` as our backend API
- `JSON Web Token (JWT)` for current user instance storage
- `localStorage` to store our JWTs on the client


<br>
<strong>In order to create the best user flow, here are a few things we'll focus on:</strong>
<blockquote>
  <ul>
    <li>Creating a decorator to protect confidential components from unauthorized users</li>
    <li>If the user was previously logged in, we'll show them their dashboard on app load</li>
    <li>If the user manually changes the route causing a page refresh, we want to auto authenticate them on the new path load</li>
  </ul>
</blockquote>
<br>

Let's first set up our example routes:

{% highlight javascript %}
let routes = (
  <Route path='/' handler={AppHandler}>
    <DefaultRoute name='home' handler={HomeHandler} />
    <Route name='dashboard' path='/dashboard' handler={DashboardHandler} />
    <Route name='login' path='/login' handler={AuthHandler} />
    <Route name='join' path='/join' handler={AuthHandler} />
  </Route>
);

export default routes;
{% endhighlight %}

For simplicity sake, we'll keep it to a few basic routes. `dashboard` will be the only <strong>protected route/component</strong> in our app, meaning in order to access it, a current user must be present; otherwise we want to redirect the client to the `login` route.
<br><br>

<strong>Creating a decorator to protect confidential components from unauthorized users</strong>

In order to shield our `dashboard` component from any unauthorized users. We need to wrap it in a custom decorator to make sure that a logged in user is present before revealing it's contents. We do that like so:

{% highlight javascript %}
export default (ComponentToBeRendered) => {  
  class ProtectedComponent extends React.Component {
    constructor(props) {
      super(props);
      // we're keeping track of the current user instance
      this.state = { currentUser: AuthStore.getCurrentUser() };
      this._updateState = this._updateState.bind(this);
    }
    componentDidMount() {
      this._unsubscribe = AuthStore.listen(this._updateState);

      // returns out of method is current user already exists
      if (this.state.currentUser) return;

      let jwt = localStorage.getItem('jwt');
      let unauthorized = !this.state.currentUser && !jwt;

      // automatically authenticates the user if a JWT is found
      if (jwt) AuthActions.autoLoginUser(jwt);

      // redirect to login page is theres no current user state or any JWT
      if (unauthorized) this.context.router.transitionTo('/login');
    }
    componentWillUnmount() {
      this._unsubscribe();
    }
    _updateState(state) {
      // once AuthActions.autoLoginUser returns success and authenticates, the current user state is updated
      this.setState({ currentUser: state.currentUser });
    }
    render() {
      let s = this.state;
      
      // we show a loading screen initally, and based on if the current user is updated or not, we either show the contents intended to be rendered, or redirec the client to the login page.

      if (s.currentUser) {
        return <ComponentToBeRendered {...this.props} currentUser={s.currentUser} />;
      } else {
        return <Spinner fullScreen={true} />;
      }
    }
  }

  ProtectedComponent.contextTypes = {
    router: React.PropTypes.func
  };

  return ProtectedComponent;
}
{% endhighlight %}

Now whenever we need to protect a component to only authorized users, we simply call it like so:

{% highlight javascript %}
class DashboardHandler extends React.Component {
  // ...
}

// here we wrap our decorator around our component
export default ProtectedComponent(DashboardHandler);
{% endhighlight %}

When a user tries to access this component either through the app's flow or manually through the URL, they will either be authenticated and saved in state or redirected to a manual login page.
<br><br>

<strong>Get or authenticate the current user when a new route is rendered</strong>

First, let's quickly configure the server side render code:

{% highlight javascript %}
app.use((req, res, next) => {
  // ...
  Router.run(routes, req.url, (Handler, state) => {
    let content = React.renderToString(<Handler />);

    res.render('index', { content: content });
  });
});
{% endhighlight %}

According to our route definitions, the default component a user hits on load is the `HomeHandler`. In order for the user to be redirected if logged in, we first must check for an existing current user instance here.

{% highlight javascript %}
export default class HomeHandler extends React.Component {
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    // when the component mounts, we want to run a method and see if there's any current user available
    AuthHelper.updateCurrentUser();
  }
  render() {
    return (
      <div>
        <h1>My App!</h1>
        <p>Welcome to the home page!</p>
      </div>
    );
  }
}
{% endhighlight %}

Here's our `AuthHelper` file:

{% highlight javascript %}
class AuthHelper {
  // checks for the store state for current user, if non existent then it auto logins based on json web token in localStorage
  updateCurrentUser() {
    if (AuthStore.getCurrentUser()) return;

    let jwt = localStorage.getItem('jwt');
    if (jwt) AuthActions.autoLoginUser(jwt);
  }
}

export default new AuthHelper;
{% endhighlight %}

If a `currentUser` state exists on the `AuthStore`, it means that the loading of the default route was likely  triggered programatically, therefore `AuthStore` state is persisted and there's no need to reevaluate the current user.

However, if the route was <strong>manually triggered</strong>, it would run server-side, meaning no `currentUser` state exists on the `AuthStore`. In that case we check for a `JWT` in `localStorage` and if such a token exists, we use that to call our API and authenticate the user. Here's our `AuthActions` and `AuthStore`:

{% highlight javascript %}
var AuthActions = Reflux.createActions({
  'autoLoginUser': { children: ['completed', 'failed'] }
});

AuthActions.autoLoginUser.listen(function(jwt) {
  let savedJwt = localStorage.getItem('jwt');
  if (savedJwt !== jwt) {
    // if the jwt passed in and the jwt in localStorage don't match, then we redirect them to the manual login page to reprompt for password

    // I will explain what RouterContainer is a little bit later
    RouterContainer.get().transitionTo('/login');
  } else {
    sendSomeAjaxRequestToAuthenticateUser(...)
    .then(this.completed)
    .catch(this.failed);
  }

module.exports = AuthActions;
});
{% endhighlight %}

{% highlight javascript %}
var AuthStore = Reflux.createStore({
  // ...
  onAutoLoginUserCompleted: function(response) {
    this._saveSessionAndRedirect(response);
  },
  _saveSessionAndRedirect: function(response) {
    this.state.jwt = response.data.token;
    this.state.currentUser = response.data.user;
    this.trigger(this.state);
    // Here we see the RouterContainer again
    RouterContainer.get().transitionTo('/dashboard');
  }
});

module.exports = AuthStore;
{% endhighlight %}

Once we call `autoLoginUser`, it makes an AJAX call to our API to authenticate the `JWT`, once the token has been verified, we save the current user data in our state, trigger it and redirect the user to their dashboard.

Here I've used `RouterContainer` twice, once in my actions and once in my store. Both times, I've used it to redirect the user from within Flux. `RouterContainer` is just a container for our instance of the the [React Router][router]. We set this initally in our client side render like so:

{% highlight javascript %}
//...

// sets the router instance in the RouterContainer, so the routes can be accessed by util classes and Reflux
RouterContainer.set(router);

router.run(routes, Router.HistoryLocation, Handler => {
  React.render(<Handler />, document.getElementById('app'));
});
{% endhighlight %}

Here's the file for `RouterContainer`:

{% highlight javascript %}
let _router;

let RouterContainer = {
  // accepts the router as a argument and sets the internal _router as the router instance passed in
  set: router => _router = router,
  // return the instance of router set on the RouterContainer
  get: () => _router
};

export default RouterContainer;
{% endhighlight %}
<br>

Once we've set the instance of router in our `RouterContainer`, we can reuse it anywhere in our app by calling `RouterContianer.get()`, including our Flux actions and stores.

Now when the user tries to access our application, either from the root URL or by manually entering a URL (ie `http://www.myapp.com/dashboard`), we can authenticate and either reveal the app content or redirect then to the appropriate page.
<br><br>

<strong>Thats it!</strong> That's the very rough and dirty guide to user authentication in an isomorphic React app.

I should mention that if you're only authenticating a client side app and there's no server side rendering, you should check out the `willTransitionTo` static method on [React Router][router] as mentioned in the [React Auth Flow][reactauthflow] blog post I read. The reason why I didn't use `willTransitionTo` is because when we render server side, the state of our stores and `localStorage` are both unaccesible, therefore it's much easier to show a loading screen for a split second while the client authenticates the user directly after the server renders the inital React code.

[reactauthflow]: https://auth0.com/blog/2015/04/09/adding-authentication-to-your-react-flux-app/
[router]: https://rackt.github.io/react-router/
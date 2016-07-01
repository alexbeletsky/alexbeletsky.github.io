---
layout: post
title: "Higher Order Components: Theory and Practice"
date: 2016-05-07 14:47
comments: true
categories: react redux
---

What I enjoy the most working with React.js is its [functional approach](http://engineering.blogfoster.com/the-functional-approach-to-ui/) for building UI's. Everything you see on a screen is essentially a component. Components are *composed* of other components, making complex interfaces possible.

Higher-order components concept goes back to [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function), *functional programming* concept, describing the function that takes other function(s) and returns a function. In exactly the same way, higher-order component takes another component(s) and return a component.

Where might it be useful? Let’s consider a few practical cases.

<!-- MORE -->

## Authentication and Authorization

Web applications have different areas, which are accessible depending on user’s authentication and access control.

There are *guest* routes, like `/signup`, `/signin`, `/forgot-password` etc, as well as *private* routes, `/dashboard`, `/user-profile` and so on.

For example,

```javascript
const routes = (
  <Route path="">
    <Route path="/signup" component={SignUp} />
    <Route path="/signin" component={SignIn} />
    <Route path="/welcome-on-board" component={Welcome} />
    <Route path="/forgot-password" component={ForgotPassword} />
    <Route path="/signout" component={SignOut} />

    <Route path="/" component={App}>
      <Route name="dashboard" path="dashboard" component={Dashboard} />
      <Route name="profile" path="user-profile" component={UserProfile} />
    </Route>

    <Route path="*" component={FourOFour} status={404} />
  </Route>
);
```

What we want to do: Everything under `/` URL is protected from unauthorized access.

One solution could be to extend `App` itself, with `componentDidMount` and `componentDidUpdate` and check if the user is authenticated. If not, redirect the user to `/signin`. It sounds right from the very beginning, but this solution has two drawbacks.

Firstly, it violates the *single responsibility* principle, what now mixes up functionality to `App` component, that originally doesn’t belong to it. And secondly, in case we want to introduce another route with similar behavior, that logic has to be present there as well, that would lead to some code duplications.

Let’s introduce a higher-order component for that,

```js
import React, { PropTypes } from 'react';
import { connect } from 'react-redux';
import { push } from 'react-router-redux';

export default function requiresAuth(Component) {
  class AuthenticatedComponent extends React.Component {
    static propTypes = {
      user: PropTypes.object,
      dispatch: PropTypes.func.isRequired
    };

    componentDidMount() {
      this._checkAndRedirect();
    }

    componentDidUpdate() {
      this._checkAndRedirect();
    }

    _checkAndRedirect() {
      const { dispatch } = this.props;

      if (!this.props.user) {
        dispatch(push('/signin'));
      }
    }

    render() {
      return (
        <div className="authenticated">
          { this.props.user ? <Component {...this.props} /> : null }
        </div>
      );
    }
  }

  const mapStateToProps = (state) => {
    return {
      user: state.account.user
    };
  };

  return connect(mapStateToProps)(AuthenticatedComponent);
}
```

We are using Redux here, but in general, the concept doesn’t depend on Redux at all.

So, `requiresAuth` is a function that takes `Component` and returns `AuthenticatedComponent`. `AuthenticatedComponent` wraps the original component, plus it checks if the user already authenticated, in case he is not, it will redirect to `/signup`.

Updated routing,

```javascript
const routes = (
  <Route path="">
    <Route path="/signup" component={SignUp} />
    <Route path="/signin" component={SignIn} />
    <Route path="/welcome-on-board" component={Welcome} />
    <Route path="/forgot-password" component={ForgotPassword} />
    <Route path="/signout" component={SignOut} />

    /* App component is wrapped with requiresAuth()  */
    <Route path="/app" component={requiresAuth(App)}>
      <Route name="dashboard" path="dashboard" component={Dashboard} />
      <Route name="profile" path="user-profile" component={UserProfile} />
    </Route>

    <Route path="/config" component={requiresAuth(Config)}>
      <Route name="page" path="page" component={PageConfig} />
    </Route>

    <Route path="*" component={FourOFour} status={404} />
  </Route>
);
```

By extending `requiresAuth` with additional parameters, it’s very easy to add some ACL functionality,

```javascript
<Route path="/app" component={requiresAuth(App, { role: 'user', redirectTo: '/signin' })}>
  <Route name="dashboard" path="dashboard" component={Dashboard} />
  <Route name="invoices" path="user-profile" component={UserProfile} />
</Route>

<Route path="/config" component={requiresAuth(Config, { role: 'admin', redirectTo: '/app' })}>
    <Route name="page" path="page" component={PageConfig} />
</Route>
```

Now, `_checkAndRedirect()` function is modified to take a role into account,

```javascript
_checkAndRedirect() {
  const { dispatch, user } = this.props;
  const { role, redirectTo }  = options;

  if (!user || !user.role === role) {
    dispatch(push(redirectTo || '/signin'));
  }
}
```

## Redirections

Another case is a redirection of a user. A redirect is forced for some users in specific conditions. For instance, we might want a user to complete some actions during his onboarding, and prevent access to a particular route, if these actions are not completed.

Otherwise, if onboarding is completed, we want to redirect a user to the dashboard.

To make it possible, we introduce `checkBoarding()` function and a corresponding `CompletedComponent`.

```js
export default function checkBoarded(Component, { withValue, redirectTo }) {
  class CompletedComponent extends React.Component {
    static propTypes = {
      state: PropTypes.object.isRequired,
      dispatch: PropTypes.func.isRequired
    };

    componentDidMount() {
      this._checkAndRedirect();
    }

    componentDidUpdate() {
      this._checkAndRedirect();
    }

    _checkAndRedirect() {
      const { dispatch } = this.props;

      if (this._shouldRedirect()) {
        dispatch(push(redirectTo));
      }
    }

    _shouldRedirect() {
      return this.props.state.user.boarded === withValue;
    }

    render() {
      return (
        <div className="boarded">
          { !this._shouldRedirect() ? <Component {...this.props} /> : null }
        </div>
      );
    }
  }

  const mapStateToProps = (state) => {
    return {
      state: state.account
    };
  };

  return connect(mapStateToProps)(CompletedComponent);
}
```

Routing,

```javascript
const routes = (
  <Route path="">
    <Route path="/signup" component={SignUp} />
    <Route path="/signin" component={SignIn} />

    // applied here..
    <Route path="/welcome-on-board" component={checkBoarded(Welcome, { withValue: true, redirectTo: '/app/dashboard'}) } />

    <Route path="/forgot-password" component={ForgotPassword} />
    <Route path="/signout" component={SignOut} />

    // .. and here
    <Route path="/app" component={checkBoarded(requiresAuth(App), { withValue: false: redirectTo: '/welcome-on-board' })}>
      <Route name="dashboard" path="dashboard" component={Dashboard} />
      <Route name="profile" path="user-profile" component={UserProfile} />
    </Route>

    <Route path="/config" component={requiresAuth(Config)}>
      <Route name="page" path="page" component={PageConfig} />
    </Route>

    <Route path="*" component={FourOFour} status={404} />
  </Route>
);
```

## Setting up context

Some routes (or more typically, group of routes), require some data context. Say, `/website/1` and `/website/1/banners`, `/websites/1/revenues` all requires that website with `id:1` is loaded and available as part of the state.

Instead of loading the same data in each component, we might use higher-order component.

```javascript
export default function websiteContext(Component) {
  class WebsiteContextComponent extends React.Component {
    static propTypes = {
      dispatch: PropTypes.func.isRequired,
      state: PropTypes.object.isRequire,
      params: PropTypes.shape({
        websiteId: PropTypes.string.isRequired
      })
    };

    componentDidMount() {
      this._setCurrentWebsite();
    }

    componentDidUpdate() {
      this._setCurrentWebsite();
    }

    _setCurrentWebsite() {
      const {
        dispatch,
        params: { websiteId },
        state: { websites }
      } = this.props;

      // prepare context data here..
      const website = _.find(websites, { id: websiteId });

      dispatch(loadContextWebsite(website));
    }

    render() {
      const { state: { website } } = this.props;

      return (
        <div className="website-context">
          { website ? <Component {...this.props} /> : null }
        </div>
      );
    }
  }

  const mapStateToProps = (state) => {
    return {
      state: state.application
    };
  };

  return connect(mapStateToProps)(WebsiteContextComponent);
}
```

Routing,

```javascript
const routes = (
  <Route path="">
    <Route path="/signup" component={SignUp} />
    <Route path="/signin" component={SignIn} />

    // applied here..
    <Route path="/welcome-on-board" component={checkBoarded(Welcome, { withValue: true, redirectTo: '/app/dashboard'}) } />

    <Route path="/forgot-password" component={ForgotPassword} />
    <Route path="/signout" component={SignOut} />

    // .. and here
    <Route path="/app" component={checkBoarded(requiresAuth(App), { withValue: false: redirectTo: '/welcome-on-board' })}>
      <Route name="dashboard" path="dashboard" component={Dashboard} />
      <Route name="profile" path="user-profile" component={UserProfile} />

      <Route name="websites" path="websites/:id" component={websiteContext(Websites)}>
        <Route name="banners" path="banners" component={Dashboard} />
        <Route name="revenues" path="revenues" component={Revenues} />
      </Route>

      </Route>
    </Route>

    <Route path="/config" component={requiresAuth(Config)}>
      <Route name="page" path="page" component={PageConfig} />
    </Route>

    <Route path="*" component={FourOFour} status={404} />
  </Route>
);
```

## Compose all the things

We sometimes need to apply a few higher-order components on the same route. As from routing above, we can see that `/app` route both `requiresAuth()` and `checkBoarded()`. Sooner or later, you might have 3-4 function calls there; that makes `Routes` a bit bulky.

Fortunately, higher-order components are easily composable,

```javascript
const CheckedWelcome = composeComponents(
  Welcome,
  [
    (c) => checkBoarded(c, { withValue: true, redirectTo: '/app/dashboard' }),
    (c) => requiresAuth(c)
  ]
);

const CheckApp = composeComponents(
  App,
  [
    (c) => checkBoarded(c, { withValue: false, redirectTo: '/welcome-on-board'}),
    (c) => requiresAuth(c),
    (c) => requiresSmth(c, { options: {} })
  ]
);

const routes = (
  <Route path="">
    // much more clean route definition
    <Route path="/welcome-on-board" component={CheckedWelcome} />

    <Route path="/app" component={CheckedApp}>
      // ...
    </Route>
  </Route>
);
```

`composeComponents()` is a simple reducer utility function,

```javascript
export function composeComponents(component, wrappers = []) {
  return wrappers.reduce((c, wrapper) => wrapper(c), component);
}
```

So, higher-order components is a great concept that aims to [replace Mixins](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.ujvvraysa) as composition mechanism. My experience showed they are great as containers wrappers, but in general, they have many practical applications.

*Originally published at [blogfoster Engineering](http://engineering.blogfoster.com/higher-order-components-theory-and-practice/).*

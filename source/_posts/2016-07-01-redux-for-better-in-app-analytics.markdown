---
layout: post
title: "Redux For Better in-App Analytics"
date: 2016-07-01 14:50
comments: true
categories: react redux javascript analytics
---

Lean startup, the term coined by Eric Ries, affected the way we build new products and services. Essentially, it says that every business idea has to be turned to a series of testable *hypotheses*. Based on these hypotheses we **build** or introduce the changes to existing products. Those changes have to be **measurable**, so we collect the results of actions we did. Based on results we prove or disapprove particular hypothesis. During the cycle, we derive **learnings**, these learnings give a foundation for a next *Build-Measure-Learn* cycle.

<!-- MORE -->

For web-based products, measurements typically mean collecting of different user behavior metrics. Based on these metrics we can see, what users are *doing* in the application, their behavior has a direct influence on a product.

Many projects I participated, had "Add Analytics" as a bottom-most task in development backlog. Ignoring the task, made it hard to implement later stages. You need something like `analytics.trackEvent('something happened', { data: 123 });` in many places, making code looks ugly and quite easy to miss some important details.


## Redux

In my [previous](http://engineering.blogfoster.com/the-functional-approach-to-ui/) article, I already mentioned the Redux similarities to [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) architecture paradigm. One of the most interesting side-effects of Event Sourcing is `Audit Log`.

Audit Log gives you an ability to look back and see what happened. As the original idea of Redux, it allows you travel back in time.

If you follow Redux rules, everything that happens in your application is represented as an `action`. In terms of analytics, it means you *already* have all events defined, you don't need to find the place where to inject `analytics.trackEvent()` code.

Take a look into this code,

```js
import { createConstants } from '../utils';

const constants = createConstants(
  // account actions
  'ACCOUNT_FORM_RESET',
  'ACCOUNT_FORM_CHANGE',
  'ACCOUNT_FORM_ERRORS',
  'ACCOUNT_FORM_ERRORS_CLEAN',
  'ACCOUNT_ACCESS_TOKEN_FETCHING',
  'ACCOUNT_ACCESS_TOKEN_FETCHED',
  'ACCOUNT_ACCESS_TOKEN_FETCH_FAILED',
  'ACCOUNT_EMAIL_CONFIRMED',
  'ACCOUNT_EMAIL_CONFIRMATION_FAILED',

  // create website actions
  'CREATE_WEBSITE_BLOG_URL_CHANGED',
  'CREATE_WEBSITE_BLOG_TITLE_CHANGED',
  'CREATE_WEBSITE_BLOG_TITLE_RESOLVE_FAILED',
  'CREATE_WEBSITE_STATE_LOADING',
  'CREATE_WEBSITE_STATE_LOADED',
  'CREATE_WEBSITE_WIZARD_STEP_NEXT',
  'CREATE_WEBSITE_WIZARD_STEP_PREV',

  // ...
```

Those are constants for action types. Our goal is to turn those action types, into meaningful analytics events.

## Analytics Server

At blogfoster, we are using custom analytics solution. Essentially it's an HTTP API, which receives an event and stores and distributes it for further processing. It makes it easy from integration perspective both for the web and mobile clients.

Even the back-end is quite complex there, the interface is rather simple,

```
HTTP POST /v1/event

{
  eventType,
  payload
}
```

## Middleware

If you have worked with [Express.js](http://expressjs.com/) or [Koa.js](http://koajs.com/) frameworks, then you should already be familiar with a *middleware* concept. Middlewares are functions that could be injected into response-request pipeline and designed to produce different side-effects (e.g. request modification, logging, redirections, performance measurements, etc.)

Redux introduces the concept of middleware as well, which makes library not only flexible but applicable for *real-world* applications. We know that Redux is essentially all about synchronous workflow, but the nature of web applications is asynchronous (events, client/server communications) and this is where middleware shines.

The API of Redux middleware is pretty elegant; it's not hard to implement own functions. If we inject the function, that can analyze the action and send data to analytics server, we will reach our goal, without touching any application logic.

```js
import { client } from '../analytics';

const handleAction = (store, next, action, options) => {
  if (!action.meta || !action.meta.analytics) {
    return next(action);
  }

  const { eventType, eventPayload } = action.meta.analytics;

  client(options).track(eventType, eventPayload);

  return next(action);
};

export function createAnalytics(options = {}) {
  return store => next => action => handleAction(store, next, action, options);
}
```

As you can see, the `handleAction()` function checks, if there is `meta` data related to `analytics` exists in action, it will pass it to `client` that would when to make HTTP request to analytics server.

## Configuring store

We need to apply middleware function now, it's done via `applyMiddleware()` Redux helper,

```js
import { createStore, applyMiddleware } from 'redux';
import { createAnalytics } from '../middleware';

const middlewares = [
  thunk,
  createLogger(),
  createAnalytics({ host: 'ANALYTICS_HOST', token: 'ANALYTICS_TOKEN' })
];

const enchancer = applyMiddleware(...middlewares);

export default function configureStore(rootReducer, initialState) {
  return createStore(rootReducer, initialState, enchancer)
}
```

## Augmenting Actions

I came up with a small utility function, to minimize modification of existing action creators as much as possible,

```js
import constants from '../../constants';

export function loadedDashboardState(website, status, news, revenues) {
  return trackable({
      type: constants.DASHBOARD_STATE_LOADED,
      payload: { website, status, news, revenues }
    },
      'Dashboard opened',      // <- event name
      { name: website.name }   // <- additional event data
    );
}

export function loadingDashboardStateFailed(err, resp) {
  return trackable({
      type: constants.DASHBOARD_STATE_LOADING_FAILED,
      payload: { err, resp }
    },
      'Dashboard loading failed',
      { error: resp.text }
    );
}
```

Where `trackable()`,

```js
export function trackable(action, event, properties = {}) {
  const analytics = {
    eventType: event,
    eventPayload: properties
  };

  return { ...action, meta: { analytics } };
}
```

That's it. Now go through all action creators and wrap actions you need to log into `trackable()` function, so they became trackable, and middleware would send them to the analitycs server.

## Recap

Even if you don't think about analytics from the very beginning, Redux architecture allows you to plug it in whenever you want (or need it). Based on `Audit Log` you can watch and turn actions into corresponding analytics events. Middleware functions would allow to hook all actions and turn them into analytical events.

**PS**. During my writing time, developers from [Rangle.io](https://github.com/rangle) participated *React-Europe* conference and gave a nice talk about the topic above. So, I would recommend you to [check it out](https://www.youtube.com/watch?v=MBTgiMLujek) for further details.

*Originally published at [blogfoster Engineering](http://engineering.blogfoster.com/redux-for-better-in-app-analytics/).*

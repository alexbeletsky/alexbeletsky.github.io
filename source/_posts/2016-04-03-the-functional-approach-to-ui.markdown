---
layout: post
title: "The Functional Approach to UI"
date: 2016-04-03 14:45
comments: true
categories: react, redux, javascript
---

A state is a representation of a system in a given time. Traditionally, we deal with *mutable* state storages, like SQL and NoSQL databases.

Each update destroys previous value, there is no history what value had been stored a minute ago, an hour ago or 5 years ago.

<!-- MORE -->

## Events and Immutability

Are there any alternatives, that allow mitigating that drawback?

One of the oldest professions in the world (not what you think) already have an answer for that. In *accounting* the only way to modify the balance is to introduce a credit or debit record.

If an accountant makes a mistake of placing *$100 debit*, she simply can’t erase an entry. Instead, she introduces a contra-operation, new entry that says *$100 credit*, that should compensate balance and return it to the previous state.

That principle makes a foundation for *Event Sourcing* architectural style. In Event Sourcing, there is an initial state and series of events, each event represents a *change* it introduces to the state. Every event is stored, and can not be deleted or modified, state is derived from events. Both are **immutable**.

The state of a whole system can be described with such formula:

```
Snext = F(S, event);
```

To get the state of the system for given moment of time, we need to take an initial state and *replay* all events that happened before.

## UI as Function of State

An imperative approach is dominant now, and front-end applications are not the exception. We are using MVC / MVVM on a client, fetching data from a server, imperatively modifying it and storing back.

The state of UI is distributed all around, there is no central place, where it’s stored. It’s rather a mix of states in models, states in DOM with a bunch of side-effects.

What if we imagine such central place and the whole UI of our application, as a function of it?

```
UI = F(S);
```

Means, UI is a function of a state.

[React](https://facebook.github.io/react/) is a library that gives the ability to treat UI like that. React represents the UI as a hierarchy of components, each of the components is a [function](https://facebook.github.io/react/docs/displaying-data.html#components-are-just-like-functions), that takes some properties as input and returns a markup. The whole UI is a composition of such functions.

But how to deal with the state to avoid the problems we already had with an imperative approach to UI?

## Predictable State

[Dan Abramov](https://github.com/gaearon), in his famous talk of [Time Travel Debugging](https://www.youtube.com/watch?v=xsSnOQynTHs) came to the idea, of bringing event sourcing principles of state handling to the client, solving one of the most painful issues.

[Redux](http://redux.js.org/) represents the state as,

```
Snext = R(S, action);
```

Combining it with UI part,

```
UI = F(S);
UI = F(R(state, action));
```

**F** - `React` component function, **R** - `Reducer`, a function that transforms the current state, based on *action* that happened in an application.

In Redux architecture, we have a *store* that holds the application state and provides an interface for changing that state, by *dispatching* actions. An *Action* holds it’s type as well as payload, which reducer uses to produce next state.

Like in Event Souring, actions are just plain objects, they can be logged, serialized, stored, and later replayed for debugging or testing purposes.

## React with Redux

Even if Redux stresses it’s not designed to be used especially with React, React and Redux make a perfect couple.

React allows you to treat the UI as a function of the state, Redux allows you to manage that state in a predictable way. [React-Redux](https://github.com/reactjs/react-redux) binding *connects* React component to Redux state, and every time it changes, component gets new state and UI re-rendering triggers.

In other words, with each change, the whole UI is re-rendered. Because of React re-rendering is based on virtual DOM diff algorithms, complete re-render doesn’t bring huge performance penalties, comparing to other frameworks.

Unidirectional data flow makes it easy to understand what’s going on in complex interfaces. As I [jumped in](http://engineering.blogfoster.com/jumpstart-to-react-redux-development/) with Redux with-out considerable experience with React itself, I would say once *state* part as a problem is solved, React part becomes a simple one.

*Originally published at [blogfoster Engineering](http://engineering.blogfoster.com/the-functional-approach-to-ui/).*
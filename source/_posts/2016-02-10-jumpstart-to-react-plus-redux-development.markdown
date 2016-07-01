---
layout: post
title: "Jumpstart to React+Redux Development"
date: 2016-02-10 14:35
comments: true
categories: react, redux, javascript
---

Somewhere November 2015, we realized that our existing product requires a proper update since it remained in a beta phase, for quite some time. We were thinking, what would be the best approach to take?

There are many things you need to consider, but primarily - business and technological risks. For companies, with an established customers base, value proposition and product, taking a risk of a massive rewrite is always very high. For startups, speed to market and responsiveness to change is a priority, so they are pivoting and rewriting products from scratch.

<!-- MORE -->

## Our approach

At [blogfoster](http://www.blogfoster.com/), we are kind of in the middle. The young company, hitting product-market fit, having customers and making revenues. We thought about a combined approach. Enhance our existing front-end application, by substituting some pieces of it, using a technological stack for further growth and development speed.

We had a common opinion about the technology that we would like to pick up. Having a good awareness of which technologies are taking off now, we had the confidence to bet on them.

We agreed on the following: [React](https://facebook.github.io/react/) for handling UI, [Redux](https://github.com/rackt/redux) for managing state, [Webpack](https://github.com/webpack/webpack) as bundler, and ES2015 with [Babel](https://babeljs.io/), as programming language. On the server side, [Hapi.js](http://hapijs.com/) for the REST API, MySQL and [Sequilize](http://docs.sequelizejs.com/en/latest/) as a data storage and access.

Since almost all of libraries and tools, mainly a front-end ones, mentioned above were new to us, our CTO [Dominic](https://github.com/dominicumbeer), proposed an approach of starting up with developing of boilerplates both for new server and client, so we build our applications based on them.

The goal was to build some simple application, which would involve quite typical web applications operations, e.g. routing, rendering, fetching and sending data to a REST API. Since signin/signup is the entrance of all applications, it makes sense to implement it as a first prototype.

I was pretty sure, I need about two days on that.

It took me a week to get it in shape so that we're confident to continue with it. React, Redux, Webpack, Babel are quite hard to combine, if you are doing it at the first time.

Honestly, I had a lot of frustration with the [boilerplate](https://github.com/blogfoster/front-stack-boilerplate) implementation. On the way, I seriously thought a few times if it is the right approach we are going to take.

## Webpack

[Webpack](https://github.com/webpack/webpack) appeared to be a huge piece of software, with very unintuitive API. A configuration of Webpack is more difficult, comparing to grunt/gulp + browserify + tools setup. I started to see Webpack as huge kitchen sink. Transpiring, bundling, optimizing, hot-reloading, server deployment - all is in the box.

But once it’s configured, you realize it is worth.

<img width="1171" alt="screenshot 2016-01-30 13 25 31" src="https://cloud.githubusercontent.com/assets/304929/12695647/182e7662-c755-11e5-9216-f2c50d18d169.png">

First of all, it works quite fast. Our application is still small, but it takes only 1.6s to fully rebuild it. Secondly, Webpack is much more than simple a build tool. It introduces you to another way of working with web resources such as sources, styles and images.

Finally, Hot Reload is something that makes development with Webpack really fun. Having the experience of updating a portion of the applications without refreshing the page is totally awesome.

## Babel and ES2015

Babel 6 has been released at the end of October. I started to use it from the very beginning, but it was quite a mistake. A lot of libraries still had dependencies on Babel 5 and refused to work properly. I had a bunch of cryptic error messages in a console, read a bunch of GitHub issues before I realized that. That forced me to step back for [react-hot-reloader](https://github.com/gaearon/react-hot-loader), instead of [react-transform](https://github.com/gaearon/react-transform-boilerplate) since it wasn’t yet ported to Babel 6 and we still have some small [issues](https://github.com/feross/standard/issues/372) after Babel updates.

But now, if you ask me - would you like to switch back to ES5, I would likely say no. I wasn’t happy when I heard the first time that ES6 will have classes. But, implementing React components as classes, makes a lot of sense.

Moreover, spread operator, fat arrow functions, and new scoping rules make programming with ES2015 as a much more pleasant process.

## Redux

[Redux](https://github.com/rackt/redux) is one of the most popular JavaScript libraries ever. Taking the functional programming approach, combined with a nice API and a great timing of the release, Redux is de-facto default Flux implementation for React-based applications.

It requires a lot of paradigm shifts, cause it’s very different from MVC/MVVM libraries I used to work with.

Redux philosophy is based on the fact, that the whole application state is represented as *one big object*. The application’s UI, becomes a *function of this* state object. The only way to change the state, is to *dispatch* an action. An action contains it’s type and some payload, which is passed to a reducer function. The reducer function produces a new instance of the state. Based on the updated state, the application re-renders their UI.

As you see, all that approach is pretty much synchronous. It was quite hard for me to understand how to introduce asynchronous behavior there. I was very happy once my [redux-thunk](https://github.com/gaearon/redux-thunk) implementation started to work.

Besides, hooking up Redux to React the application was not super straightforward. I spent some time with an understanding of the configuration of the storages, connected components, etc.

I’m still having a lot of questions. I not really happy with all the code you need to write if you want to introduce new actions. And I don’t know yet how to test [properly](http://stackoverflow.com/questions/33997081/react-redux-loading-application-state-in-component-constructor) all that stuff.

But Redux gives a nice tooling around, plus better confidence of what’s going on in the application. Since all actions could be logged, and state never overwrote it’s much more easy to debug and introduce new functionality.

## React

0.14 is the latest React release, released in October.

First of all, it *just works*. I feel so great dealing with React, since it brings so less surprises. The concept of a component is simple and very native.

I’m not sure yet if I use everything right. Designing the component hierarchy is still a question, that requires a bit more practice.

I’m happy with the error messages that React provides. The messages pointing you directly to the exact problem. Compared to other libraries, which produce nothing more as encrypted error messages (yes, [angular](https://daveceddia.com/angular-2-errors/)). Making UI's with React feels great.

Currently, we started to involve our designers to work directly with JSX instead of HTML and we are looking forward to see the impact there.

<img width="1287" alt="screenshot 2016-01-30 13 37 55" src="https://cloud.githubusercontent.com/assets/304929/12695696/b561681c-c756-11e5-99e6-0c73627720f9.png">

Second, React is likely the biggest change to front-end development in the last 5 years. Virtual DOM, diff tree and partial re-rendering - quite a masterpiece library. I can’t say it feels totally perfect for me, but it feels really good.

## ESLint

I have to mention ESLint. Previously, I used JSHint, but unfortunately, they decided not to support latest language features until official support.

But ESLint actually becomes so much nicer. It so simple to configure and it’s so helpful, that I can not imagine to develop without it anymore.

<img width="975" alt="screenshot 2016-01-30 13 41 17" src="https://cloud.githubusercontent.com/assets/304929/12695714/2d8eca14-c757-11e5-905f-a66065111587.png">

Both Atom and Sublime have ESLint support, and it will be the only choice if you want to go the ES6/React/Webpack way with linting.

## Conclusions

After the [boilerplate](https://github.com/blogfoster/front-stack-boilerplate) has been concluded, we started with the implementation of our new application based on that. It works fine so far, not ideally, but good enough. During the flight, we skipped our idea of changing pieces of existing app and started new project from scratch.

Greenfield development + latest libraries and tools make whole development process nice and fun.


*Originally published at [blogfoster Engineering](http://engineering.blogfoster.com/jumpstart-to-react-redux-development/).*
---
layout: post
title: "Seismo - Analytics For Your Startup"
date: 2013-12-05 15:32
comments: true
categories: analytics startup
---

Sometimes ago I [wrote the post](http://beletsky.net/2013/07/think-ahead-think-logging.html), where was thinking about importance of logging of application state to clearly see what's going on inside and react accordingly. Logging is vital for any reliable system.

If *logging* is a must from development point of view, *analytics* is a must from business point of view. You would like to see, how many users signs-in and signs-up during the day, what actions they do inside the app, what issues they use all the time, what issues they never touch.

Following "Invent Own Bicycle" principle, I've created small project to attack the problem - [Seismo](https://github.com/likeastore/seismo).

<!-- More -->

## Overview

Right now, [seismo](https://github.com/likeastore/seismo) repository is a bunch of javascript files and I'm going to decouple project a bit to have it in more structured way. But in essence, there is REST API, with token-based authentication model, which allows you to push application events, store them to MongoDB and then build reports on those events.

To simplify the integration, there are [seismo-node-client](https://github.com/likeastore/seismo-node-client). It's [request](https://github.com/mikeal/request)-based application, ready to use from Node.js backends. Very soon, I'm going to add [seismo-browser-client](https://github.com/likeastore/seismo-browser-client) to be used in browser.

To allow the deployment be as easy as possible, I want to pack seismo server as [docker](http://www.docker.io/) image and put it to public [index](https://index.docker.io/), so it could be deployed on any Linux machine with few seconds.

## Other languages support

For now, it has good support for JavaScript and Express.js platform. I wish to have support for other platforms like, Ruby, Python, .NET and Java. It's important not only to support particular languages, but support particular platform - so, we have good examples and reusable components for RoR, ASP.NET MVC or Spring.

## Dashboard

Data is useless without visualization. Seismo project includes [seismo-dashboard](https://github.com/likeastore/seismo-dashboard).

It's pure client side application, build with Yeoman/Angular.js and could be deployed to any static server, it works great on [gh-pages](http://pages.github.com/) as well.

<img src="/images/blog/seismo-dashboard.png" alt="seismo dashboard" class="no-shadow" />

At the moment, the dashboard is not flexible at all. But I wish to create it fully customizable and widget based.

## Conclusion

Even if we already used that for [Likeastore](https://likeastore.com/) the project is far away from being generally used. The project currently is nothing more as prototype now, but I would like to improve it in nearest future, so I hope something interesting might came out.
---
layout: post
title: "Logs Driven Development"
date: 2014-01-25 13:12
comments: true
categories: tdd logs practices
---

One of the components I currently work on called `collector` and it has no tests. Collector is all about of building queue of tasks, executing them and store data to database after. Tasks are HTTP clients that requests API and process transform responses into generic forms.

I've started it with tests using [nock](https://github.com/pgte/nock) component to mock HTTP requests, but quickly I found those tests both hard to write and no real benefit cause real responses that could broke it differs from ones I mock inside the tests.

In the same time, I regularly change that component and after changes and deployments I pretty quickly see the regressions and non-expected behavior, cause all information I need is inside of application logs. I call that "Logs Driven Development".

<!-- More -->

The key point of Logs Driven Development is then you know your logs so good, so you easily detect patterns and performance indicators, so if anything goes wrong, it became obvious. When reading logs of your application became a habit it works really well.

It probably have some similarities with [Characterization test](http://en.wikipedia.org/wiki/Characterization_test), where the behavior of application is clearly projected into logs. There are tools like [Approval Tests](http://approvaltests.sourceforge.net/) that utilize that kind of testing and automate them.

That's of cause could not be taken seriously as recommended practice since it doesn't scale well.


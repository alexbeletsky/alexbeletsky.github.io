---
layout: post
title: "Integration or Unit Tests Trade-Off"
date: 2014-04-23 11:19
comments: true
categories: nodejs expressjs tdd
---

Recently, I've released small Express.js extension for easy switching application to `maintenance` mode. Sometimes, you just want to run patch against database or change the infrastructure of product, but instead of showing blank `nginx` 503 error page, you want to have nice looking HTML, saying will be back soon.

The `maintenance` package is now available on [npm](https://www.npmjs.org/package/maintenance) and you welcome to use it. But I would like to share the way I developed and test it.

<!-- More -->

So, `maintenance` is a function that takes `app` instance and `options` as a parameters. Then it augments `app` with additional endpoint (if there is such preference) and injects middleware function that would render maintenance page in case of mode is set to `true`.

It's really small piece functionality, but still I wanted to make sure, it's gonna work in my application without annoying restarts of servers and debugging the stuff. And TDD is right approach to solve the pain.

TDD is quite commonly trade-off of `unit tests` and `integration tests`. And it's always up to developer which direction go in particular case. Let's take a look on my case:

If you go `unit test` way, it would be something like that:

* call `maintenance` function, pass mock of `express` instance
  * verify that new `route` is added after function completes
  * verify that all `route.callbacks` now have callback to check the mode
  * verify that `route.callbacks[0]` contains mode check function
* call `route.callbacks[0]`, mock `req` and `res` object
  * verify that `res.render` called with `maintanance.html` argument
  * etc..

Is all of that kind of **suck**? It is, since it would take too much effort for mocking the stuff.. But more important, all of that is nothing more as just testing of **implementation details** of Express.js, but not **behavior details** of application.

Now, let's consider `integration tests` way for same stuff:

* run application in normal mode
  * send HTTP GET and make sure that response is fine
* run application is maintenance mode
  * send HTTP GET and make sure that response contains maintenance page
* run application in normal mode
  * send HTTP GET and make sure that response is fine
  * sent HTTP POST to maintenance endpoint
  * send HTTP GET and make sure that response contains maintenance page

Does it look like real **behavior** testing? Indeed, test acts as `user` and checks that application actually behaves right, doesn't matter of implementation details.

With spending more time with Express.js and Node.js I see much [more value](http://beletsky.net/2014/03/testable-apis-with-node-dot-js.html) in integration way of testing. It's very easy to spin-up a server and send HTTP requests and check responses.

If you are interested, check out [specification](https://github.com/likeastore/maintenance/blob/master/test/spec/maintenence.spec.js) and [test application](https://github.com/likeastore/maintenance/blob/master/test/app/app.js).


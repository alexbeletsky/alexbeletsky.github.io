---
layout: post
title: "Catch Errors in Express.js Application"
date: 2013-10-18 09:56
comments: true
categories: nodejs expressjs errors
---

This is a small follow up for my [previous post](http://beletsky.net/2013/10/securing-express-dot-js-http-endpoints.html), using the same technique not for authorization, but rather for error handling.

Let's go back, to the problem. I want to handle all errors in my application. Instead of `res.send()` or `res.json()`, I want to have a middleware that handles everything by itself. It can be flexible, so I can put any kind of logic there, like logging etc.

It's very easy to archive with *patch the middleware* method.

<!-- More -->

Just like in previous case, `app.use()` won't work here. First, it would apply to every request. Second, error handling middleware have to placed last, `app.use()` won't guarantee that.

## Follow the style

To get benefits of common error handling/logging code, you have to follow particular style. It's very simple, though.

Your last endpoint (middleware) function have to always receive `next()` callback parameter, all logs have to pass as first argument to that function. You should not send errors directly as `res.send(500, 'Error')`.

```js
app.get('/api/users/:id', function (req, res, next) {
	db.users.find({id: req.params.id}, function (err, user) {
		if (err) {
			return next({message: 'failed to query db', status: 500});
		}

		if (!user) {
			return next({message: 'user not found', status: 404});
		}

		res.json(user);
	});
});
```

Please note, that it receives first argument.. and the function is only called, the we call `next()` with first parameter.

## Error handler middleware

Let's define the function. Since HTTP API's are JSON based, it would just return the JSON response and status.

```js
function logErrors(err, req, res) {
	req.unhandledError = err;

	var message = err.message;
	var error = err.error || err;
	var status = err.status || 500;

	res.json({message: message, error: error}, status);
};
```

## Apply the patch

Again, right after all routes are already defined, let's call `applyErrorLogging()`.

```js
// ...

require('./source/api')(app);
require('./source/router')(app);

applyAuthentication(app, ['/api']);
applyErrorLogging(app);					// apply error handling here

// ...
```

And `applyErrorLogging()` function,

```js
var middleware = require('../middleware');

function applyErrorLogging(app) {
	for (var verb in app.routes) {
		var routes = app.routes[verb];
		routes.forEach(patchRoute);
	}

	function patchRoute (route) {
		route.callbacks.push(middleware.errors.logErrors);
	}
}

module.exports = applyErrorLogging;
```
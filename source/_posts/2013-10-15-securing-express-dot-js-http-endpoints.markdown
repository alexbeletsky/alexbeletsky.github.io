---
layout: post
title: "Securing Express.js HTTP Endpoints"
date: 2013-10-15 21:54
comments: true
categories: nodejs expressjs security authorization
---

Once you implement HTTP API using Express.js (which is great option, btw), the security became the concern. There are a lot of diffent options and strategies, implementing security for API's. One of the latest I prefer is described here.

Doesn't matter what the actual strategy is, you have to apply it somehow in your application. In general, HTTP API security goes down to authorization issues. That would mean, you have a piece of information in HTTP request (either field in header or cookie), by checking one you can say, is this HTTP request authorized or not.

That "check" have to applied for all HTTP calls, except ones explicitly mentioned as "guest".

<!-- More -->

## Middleware

Such type of job is ideal for middleware. In fact, you might have middleware function, that does authorization check:

```js
function access(req, res, next) {
	checkAuthorization(req, function (err, authorized) {
		if (err || !authorized) {
			res.send({message: 'Unauthorized', status: 401});
		}

		next();
	});

	function checkAuthorization(req, callback) {
		// actual auth strategy goes here..
	}
}
```

Of cause, it's simply possible to apply this function to each HTTP method in application, like

```js
function peopleApi(app) {
	app.get('/api/people',
		middleware.access,
		getPeople);

	app.get('/api/people/:id',
		middleware.access,
		getPerson);

	app.post('/api/people',
		middleware.access,
		postPerson);

	// ...
}

module.exports = peopleApi;
```

but, it's really get annoying to do that all the time.. and it's easy to just forgot to secure endpoint. So, it's better to keep the code as clean as possible.

```js
function peopleApi(app) {
	app.get('/api/people',
		getPeople);

	app.get('/api/people/:id',
		getPerson);

	app.post('/api/people',
		postPerson);

	// ...
}

module.exports = peopleApi;
```

That seems to be like `app.use()` is good candidate to place `access` function into, but it's not. `app.use()` is global apply of middleware function, so if applications serves static resources, that does not need authentication, or simply you want to expose some **guest** endpoints.

## Guest or not?

Guest endpoints are ones, that can be accessed without authentication. That's a kind of special case, but typically required on any HTTP API projects I worked.

We need to distinguish between secure and guest endpoints. We'll introduce special middleware function, for guest access.

```js
function guest(req, res, next) {
	req.guestAccess = true;
	next();
}

```

Now, assume that all endpoints are require authentication by default (which is good assumption), but ones that don't need to have `guest()` middleware be used.

```js
function peopleApi(app) {
	app.get('/api/people',
		getPeople);

	app.get('/api/people/:id',
		getPerson);

	app.post('/api/people',
		postPerson);

	app.get('/api/people/meta',
		middleware.guest,				// no authentication required!
		getPeopleMeta);

	// ...
}

module.exports = peopleApi;
```

## Patch the routes

After I got a bit deeper with structure of Express.js `application` I came up to one idea that helped to solve the problem.

Then all endpoints are defined, `application` would contain initialized [routes](http://expressjs.com/api.html#app.routes) object. If you closer, than you'll see, besides of path and method data it also contains an array of `callbacks` applied to route. That's exactly middleware functions, so we can simply patch that array with authentication function we want.

The authentication function have to be called first, so it's need to be placed at `0` position of array.

Right after application configured and all routes are defined, call `applyAuthentication()` function.

```js
var app = express();

app.configure(function(){
	// configure
});

app.configure('development', function() {
	// configure for development
});

app.configure('production', function() {
	// configure for production
});

require('./source/api')(app);
require('./source/router')(app);

applyAuthentication(app, ['/api']);		// apply authentication here

http.createServer(app).listen(app.get('port'), function() {
	console.log('app listening on port ' + app.get('port'));
});
```

And finally, `applyAuthentication` function,

```js
var _ = require('underscore');
var middleware = require('../middleware');

function applyAuthentication(app, routesToSecure) {
	for (var verb in app.routes) {
		var routes = app.routes[verb];
		routes.forEach(patchRoute);
	}

	function patchRoute (route) {
		var apply = _.any(routesToSecure, function (r) {
			return route.path.indexOf(r) === 0;
		});

		var guestAccess = _.any(route.callbacks, function (r) {
			return r.name === 'guest';
		});

		if (apply && !guestAccess) {
			route.callbacks.splice(0, 0, middleware.access.authenticatedAccess());
		}
	}
}

module.exports = applyAuthentication;
```
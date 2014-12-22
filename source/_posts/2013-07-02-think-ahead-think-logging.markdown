---
layout: post
title: "Think Ahead, Think Logging"
date: 2013-07-04 09:02
comments: true
categories: nodejs expressjs javascript
---

When we develop application, we have everything to understand applications behavior. Debugger, traces, tests - all information just in hands. If something goes wrong, it's not so hard to track the problem.

Situation is completely different then app leaves development box and goes to production. In best case, we'll receive email or tweet from user, but typically problem remains on production *silently*, while customers just *silently* leave.

Prepare application to production, means prepare good error logging. I'm going to show how to extend your Express.js with proper logs.

<!-- More -->

## What it means to have good logs?

In my perspective good logs are ones satisfying following criterias:

* All unhandled errors are logged
* Log records are comprehensive and clear
* Logs are easily accessible
* If critical error logged, developers have to be notified

## Logger

Logger is object responsible to take some message or object and log it. The example of one,

```js
var util = require('util');
var colors = require('colors');
var moment = require('moment');

var logger = {
	colorsMap: {
		'success': 'green',
		'warning': 'yellow',
		'err': 'red',
		'info': 'grey'
	},

	success: function (message) {
		this.log('success', message);
	},

	warning: function (message) {
		this.log('warning', message);
	},

	error: function (message) {
		this.log('err', message);
	},

	info: function (message) {
		this.log('info', message);
	},

	log: function (type, message) {
		var record = this.timestamptMessage(util.format('%s: %s', type.toUpperCase(), this.formatMessage(message)));
		console.log(record[this.colorsMap[type]]);
	},

	formatMessage: function (message) {
		return typeof message === 'string' ? message : JSON.stringify(message);
	},

	timestamptMessage: function (message) {
		return util.format('[%s] %s', moment(), message);
	}

};

module.exports = logger;
```

`Logger` could be used everywhere you need to get some info. But our ultimate goal is to be aware of all errors might appear in application.

## Augmenting Express.js application with logs

We never know then error might appear. But, we can catch all unhandled errors + if some web request failed to complete with success code, we have to log that as well.


### Handling "unhandled" errors

We can listen to process 'uncaughtException' event. It just placed in `app.js` file. The best place is just after require section and before any object is used.

```js
process.on('uncaughtException', function (err) {
	logger.error({msg:'Uncaught exception', error:err, stack:err.stack});
});
```

From official docs,

> Emitted when an exception bubbles all the way back to the event loop. If a listener is added for this exception, the default action (which is to print a stack trace and exit) will not occur.

So, we just redirecting that error to logger. Also, docs say following:

> Don't use it, use [domains](http://nodejs.org/api/domain.html) instead. If you do use it, restart your application after every unhandled exception!

I still not switched to domain version for that, need to consider that advice.

Anyway, this `uncaughtException` will give us only information typically about `undefined` variables used, that's pretty simple to caught during development testing. More interesting stuff is what's actually happening on runtime, while application handling HTTP requests.

### Logging failing HTTP requests

Express.js power feature is *middleware*. It's possible to do a lot of cool stuff based on middleware functions. We'll utilize that feature to create a few middleware function that would allow to log all failed HTTP requests.

First one,

```js
// have to be injected as last middlware function for all routes
function logErrors () {
	return function logErrors(err, req, res, next) {
		req.unhandledError = err;

		next(err);
	};
}
```

Second one,

```js
function logHttpErrors () {
	return function logHttpErrors (req, res, next) {
		var end = res.end;
		res.end = function (data, encoding) {
			var status = res.statusCode;
			var message = {
				url: res.req.url,
				headers: res.req.headers,
				status: status,
				body: req.body,
			 	params: req.params
			};

			if (req.unhandledError) {
				message.error = req.unhandledError;
			}

			if (warning(status)) {
				logger.warning(message);
			}

			if (error(status)) {
				logger.error(message);
			}

			end.call (res, data, encoding);
		};

		next();
	};

	function warning (status) {
		return status >= 400 && status < 500;
	}

	function error (status) {
		return status >= 500;
	}
}
```

Look a bit closer: `logError()` produces middleware function that expected to be the last in chain, and if previous function retured an error, it stores that error object in in requests. `logHttpErrors()` produces middleware function that would override response `.end()` function and logs warning or error, depending on response status code.

Let's integrate to app.

`logHttpErrors()` could be put into `app.configure()` function,

```js
app.configure(function(){
	app.set('port', process.env.VCAP_APP_PORT || 3001);
	// ...
	app.use(middleware.errors.logHttpErrors());
	// ...
});

```

It's a bit more trickier with `logError()` function. As I said above, it have to be **last** callback in chain.

So, it's only possible to apply it in `app.configure()` since the routes are not defined yet. Even it's possible to manually add it to each endpoint manually, I don't think it's good idea, because it's easy to forgot do that.

I came up to following solution,

```js
require('./source/api')(app);
require('./source/router')(app);

// here .logError() will be added to end of chain
applyErrorLogging(app);

http.createServer(app).listen(app.get('port'), function() {
	var env = process.env.NODE_ENV || 'development';
	logger.info('Likeastore app listening on port ' + app.get('port') + ' ' + env + ' mongo: ' + config.connection);
});
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
		route.callbacks.push(middleware.errors.logErrors());
	}
}

module.exports = applyErrorLogging;
```

Now, it's everything in place, so all `4xx` are logged as warnings, all `5xx` are logged as errors.


## Move your logs to cloud

Simply logging information is not enough. While your application writes info to console on production machine, this information worthless to you. You have to put you logs to the place where is easily accessible.

There are few services like that. One of I recently hooked with in [Logentries](https://logentries.com/).

Logentries gives you API to submit information there + Dashboard, there logs can be viewed, search and analyzed.

![logentries dashboard](/images/blog/logentries-screen.png)

Install `node-logentries` client,

```bash
$ npm install node-logentries --save
```

And now, we need to update logger, to not only `console.log` but send it to Logentries.

Will create Logentries client,

```js
var log = logentries.logger({
	token:process.env.LOGENTRIES_TOKEN
});
log.level('info');

```

Will extend existing logger and override current `.log()` function:

```js
var logentriesLogger = (function (_super) {
	var child = {
		log: function (type, message) {
			_super.log(type, message);
			log.log(type, message);
		}
	};

	return _.extend(Object.create(_super), child);
})(logger);

module.exports = logentriesLogger;
```

Checkout this [gist](https://gist.github.com/alexbeletsky/5921464) where you can see all things put together.

So, now wherever `logger` is used, logs will both shown to screen (which is cool for development) and submitted to Logentries (which is cool for production).

## Setup notification on critical errors

If error appeared on production, developers attention should be there. Without good notification system, is too easy to skip the moment then error arises.

Again, it's easy to do with Logentries. Just go to `Alerts` section and setup patterns of errors you interested and email addresses for notifications.

![logentries alerts setup](/images/blog/logentries-alerts.png)

Email is not only one option, you can setup for SMS or webhook for your app. So, anytime error or warning appeared you will be notified and take action on it.

## Conclusions

I've used that for [likeastore](http://likeastore.com) app I currently working on and it works just fine. Having such logs gave a lot of information after we went to private-beta phase. When you see how application behaves then real users start to use, it gives you good insights about fixes and improvements to apply.

Taking into account that approach above is very universal and easy to adopt, to it today.

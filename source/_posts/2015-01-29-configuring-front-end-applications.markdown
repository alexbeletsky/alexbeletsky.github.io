---
layout: post
title: "Configuring Front-End Applications"
description: The straightforward and framework-agnostic approach for applications configuration.
date: 2015-01-29 19:42
comments: true
categories: angularjs javascript frontend browserify
facebook:
 image: https://lh5.googleusercontent.com/-Xw_YeiW-cKU/VMulmNJioWI/AAAAAAAAi48/aj98393M5YY/w2236-h1138-no/Alexander%2BBeletsky%2Bs%2Bdevelopment%2Bblog.png
twitter_card:
 image: https://lh5.googleusercontent.com/-Xw_YeiW-cKU/VMulmNJioWI/AAAAAAAAi48/aj98393M5YY/w2236-h1138-no/Alexander%2BBeletsky%2Bs%2Bdevelopment%2Bblog.png
---

Typically front-end applications have particular configuration, depending on environment. It could be access tokens, API URL's, applications settings etc. For quite long period of time I solved that problem by exposing `window.env` variable, populated either by server rendering or by plugins as `html-build` .. or just directly referencing `<script src="config/my.env.js">`, where `my.env.js` needed to be updated before actual deployment.

Spending much time on backend and working with Node.js/CommonJS I liked simplicity of `config` pattern and wanted to reuse that pattern on frontend. It's really straightforward and framework-agnostic approach.

<!--MORE-->

## Config pattern

Config pattern is something I [frequently](https://github.com/likeastore/notifier/blob/master/config) [use](https://github.com/likeastore/jobber/tree/master/config) for Node.js apps. It simply the folder with `index.js` file, containing such code:

```js
var env = process.env.APP_ENV || 'development';

var config = {
	development: require('./development.config'),
	production: require('./production.config'),
	staging: require('./staging.config')
};

module.exports = config[env];
```

The folder contains such files, `development.config.js`, `staging.config.js`, `production.config.js` etc.

The config files, simply export the object with configuration,

```js
var config = {
	connection: 'mongodb://localhost:27017/notifierdb',
	accessToken: '1234',

	hook: {
		url: 'http://localhost:5000/api/notify/',
		token: 'fake-hook-token'
	},

	logentries: {
		token: null
	},

	transport: {
		// ...
	}

	// ...
};

module.exports = config;
```

The module actually needed in configuration would `require` the `config` folder, for instance:

```js
var request = require('request');
var config = require('../config');

function fetchActions(user, callback) {
	var apiUrl = config.api.url;
	var token = config.api.token;

	request({url: apiUrl + '/actions/' + user, headers: {token: token}, callback);
}

module.exports = fetchActions;
```

## Browserify

[Browserify](http://browserify.org/) is a pre-processor (or call it transpiler) for javascript. It embraces CommonJS style of modularity and brings popular `npm` modules on client site. It's simply awesome project (many thx to [substack](https://twitter.com/substack) for it) that makes a lot of sense to me.

With Browserify you can write browser javascript in the same way you do on backend with Node.js. It implements `require()` call and bundles the application consists on different modules into one, executable in browser.

Usually, `browserify` is not run manually, but rather by means of JS build tools as `grunt` or `gulp`.

I'm going to adopt Node.js `config` pattern for frontend with browserify.

## Prepare the configuration

By analogy of node applications, let's create a `config` folder inside our `js` folder.

![config structure](https://lh6.googleusercontent.com/-tO5KVRhVjM8/VMqJVRGXiFI/AAAAAAAAi4o/2aWT3zOkNhk/w2236-h1132-no/Screenshot%2B2015-01-29%2B20.21.58.png)

`index.js` file is exactly the same as mentioned above, config files contains application dependent settings.

With example of Angular.js application, here is the `service` that used as data provider,

```js
var config = require('../config');

function drivers ($http) {
	var apiUrl = config.api.url;

	return {
		fetch: function () {
			var url = apiUrl + '/api/drivers';

			return $http.get(url);
		}
	};
}

module.exports = drivers;
```

If you just try that, it would work as charm.. but only for `development` environment.

## Setting up NODE_ENV

The problem though, `browserify` have no idea what to put into `process.env.NODE_ENV` variable, since it is `undefined` - `development` configuration is always selected.

Fortunately, `browserify` architecture supports, so called `transforms`, middleware components capable to customize browserify behaviour. One of handly transform function is [envify](https://github.com/hughsk/envify) by [Hugh Kennedy](https://github.com/hughsk).

What `envify` allows to do is basically replacement of `process.env.NODE_ENV` to particular value, eg.

```js
if (process.env.NODE_ENV === "development") {
  console.log('development only')
}
```

with `NODE_ENV` set to "production", will appear as

```js
if ("production" === "development") {
  console.log('development only')
}
```

This is exactly what we need, to make `config/index.js` work properly.

Since I use `grunt`, I'll show how to integrate `envify` in the workflow (approach for `gulp` would be similar),

```js
var envify = require('envify/custom');

module.exports = function (grunt) {
	grunt.initConfig({
		browserify: {
			dev: {
				files: {
					'source/build/app.js': ['source/js/app.js']
				},
				options: {
					browserifyOptions: {
						debug: true
					},
					transform: [envify({
						NODE_ENV: 'development'
					})]
				},
			},
			stage: {
				files: {
					'source/build/app.js': ['source/js/app.js']
				},
				options: {
					transform: [envify({
						NODE_ENV: 'staging'
					})]
				}
			},
			prod: {
				files: {
					'source/build/app.js': ['source/js/app.js']
				},
				options: {
					transform: [envify({
						NODE_ENV: 'production'
					})]
				}
			}
		},
	// ...
}
```

Now, once the `grunt build:dev` or `grunt build:prod` is run, it will populate correct `NODE_ENV` value and the rest of transpiled application would work as expected.
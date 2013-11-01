---
layout: post
title: "Using Angular.js with Require.js"
date: 2013-11-01 11:36
comments: true
categories: angularjs requirejs frontend javascript
---

I got used to idea of AMD quite long time ago. That time Require.js was best (and probably only one) good implementation that supports it. It worked great for me while I was involved into Backbone.js development. So, once I jumped in to Angular.js my first wish was reuse the same experience as previously.

There was a few difficulties with that.

<!-- More -->

## Why to use Require.js with Angular.js?

Some people argue about rationality of using Require and Angular together. Indeed, Angular has it's own module system, dependency resolve system etc. I agree with that, but still my point is: Require.js comes with very handy add-on, called `r.js` - it's code minimizer and optimizer.

Having `grunt` build system (which is de-facto standard for JS applications) you can easily integrate with your deployment scenarios.

## Project organization

Before deep dive, I want to share you some simple ideas of client side code organization that I think makes sense:

```plain
		/
		+ - build
		|
		+ - components
		|
		+ - js
			|
			+ - controllers
			|
			+ - directives
			|
			+ - services
			|
			+ - app.js
				|
				main.js
				|
				config.js
```

* **build** - results of build .js and .css is placed here (these files are used in production).
* **components** - bower components are configured to be placed in this folder.
* **js** - source code of application, divided to `contollers`, `directives` and `services`.

### main.js

Require.js main file, that contains configuration and main entry.

```js
require.config({
	paths: {
		'angular' : '../components/angular/angular',
		'ngResource': '../components/angular-resource/angular-resource',
		'ngCookies': '../components/angular-cookies/angular-cookies',
		'ngProgressLite': '../components/ngprogress-lite/ngprogress-lite'
	},
	shim: {
		ngResource: {
			deps: ['angular'],
			exports: 'angular'
		},
		ngCookies: {
			deps: ['angular'],
			exports: 'angular'
		},
		ngProgress: {
			deps: ['angular'],
			exports: 'angular'
		},
		angular: {
			exports : 'angular'
		}
	},
	baseUrl: '/js'
});

require(['app'], function (app) {
	app.init();
});
```

### app.js

Instance of application, where all Angular.js bootstraping taking place.

```js
define(function (require) {
	'use strict';

	var angular = require('angular');
	var services = require('./services/services');
	var controllers = require('./controllers/controllers');
	var directives = require('./directives/directives');

	var app = angular.module('likeastore', ['services', 'controllers', 'directives']);

	app.init = function () {
		angular.bootstrap(document, ['likeastore']);
	};

	app.config(['$routeProvider', '$locationProvider', '$httpProvider',
		function ($routeProvider, $locationProvider, $httpProvider) {
			$httpProvider.responseInterceptors.push('httpInterceptor');

			$routeProvider
				.when('/', { templateUrl: 'partials/dashboard', controller: 'dashboardController' })
				.when('/inbox', { templateUrl: 'partials/dashboard', controller: 'dashboardController' })
				.when('/facebook', { templateUrl: 'partials/dashboard', controller: 'facebookController' })
				.when('/github', { templateUrl: 'partials/dashboard', controller: 'githubController' })
				.when('/twitter', { templateUrl: 'partials/dashboard', controller: 'twitterController' })
				.when('/stackoverflow', { templateUrl: 'partials/dashboard', controller: 'stackoverflowController' })
				.when('/search', { templateUrl: 'partials/dashboard', controller: 'searchController' })
				.when('/settings', { templateUrl: 'partials/settings', controller: 'settingsController' })
				.when('/ooops', { templateUrl: 'partials/dashboard', controller: 'errorController' })
				.otherwise({ redirectTo: '/' });

			$locationProvider.html5Mode(true);
		}
	]);

	app.run(function ($window, auth, user) {
		auth.setAuthorizationHeaders();
		user.initialize();
	});

	return app;
});
```

### config.js

Configuration object that contains application settings, like `itemsPerPage`, `logoutTimeout` etc.

```js
define(function (require) {
	'use strict';

	return {
		dashboard: {
			limit: 30
		}
	};
});
```

## Master HTML

The first noticable difference is the way master HTML file is organized. You are no longer define application with directive `<html lang="en" ng-app>`, but instead using manual *booting* of application.

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
</head>
<body>
	<div class="main-viewer" ng-view></div>

	<script type="text/javascript" src="/components/requirejs/require.js" data-main="/js/main.js"></script>
</body>
</html>
```

## Booting application

Once master HTML is loaded, Require.js will execute `main.js` file. This is were the action should be taken to bootstrap Angular.js application.

Take a look at `main.js` file from above, here we start application bootstrapping:

```js
require(['app'], function (app) {
	app.init();
});
```

The `init()` function of application,

```js
var app = angular.module('likeastore', ['services', 'controllers', 'directives']);

app.init = function () {
	angular.bootstrap(document, ['likeastore']);
};
```

By doing this, you should be able to start the application and make sure that all modules, controllers and directives are loaded without any issues.

## Build configuration

Next tricky part is to configure application for production. There is a great [blog post](http://tech.pro/blog/1639/using-rjs-to-optimize-your-requirejs-project) of using `r.js` to optimize the project.

In short, you need to add [grunt-contrib-requirejs](https://github.com/gruntjs/grunt-contrib-requirejs) grunt task and configure it.

The problem with Angular.js optimization is that it using dependency injection mechanism, which resolves the services to inject by it's name. If optimizer would change the name of function parameters, when application would not work. Fortunately, there is workaround for that.

```js
requirejs: {
	js: {
		options: {
			uglify2: {
				mangle: false
			},
			baseUrl: "public/js",
			mainConfigFile: "public/js/main.js",
			name: 'main',
			out: "public/build/main.js",
			optimize: 'uglify2'
		}
	}
},
```

You have to use `uglify2` optimizer, that has an option [mangle](https://github.com/mishoo/UglifyJS2#mangler-options) that have to set to `false`. In this case, `uglify2` will do full optimization and minification of javascript code, without ruing functions parameters names, with is vital for Angular.js.
---
layout: post
title: "Simple Authentication for Angular.js App"
date: 2013-11-22 13:51
comments: true
categories: javascript angularjs frontend
---

So, you are building pure client side application that works against REST API. The client and server are completely decoupled and typically deployed separately of each other.

API's have one or another way of authenticating it's users. It could be some simple flows, like basic authorization or more complex ones as OAuth/OAuth2. But at the very end you have `token` that placed either as cookie value or HTTP request header parameter. API is then responsible to check the token for validity and if it's not valid respond with 401.

<!-- More -->

## Configure the Routes

First we need to have `/login` router where user is redirected in case of unauthorized access.

```js
'use strict';

var app = angular.module('dashboardApp', [
	'ngCookies',
	'ngResource',
	'ngSanitize'
]);

app.config(function ($routeProvider, $locationProvider, $httpProvider) {
	$httpProvider.responseInterceptors.push('httpInterceptor');

	$routeProvider
		.when('/', { templateUrl: 'views/dashboard.html', controller: 'dashboard' })
		.when('/login', { templateUrl: 'views/auth.html', controller: 'auth' })
		.otherwise({ redirectTo: '/' });

	$locationProvider.html5Mode(true);
});

app.run(function (api) {
	api.init();
});
```

## Authentication Controller and View

The authentication controller is simple module. It's responsible for sending user credentials to server and handle the response. If server authenticates user, it would return the value of access token in `.token` attribute. Otherwise, user have to be notified that something went wrong.

Btw, in [likeastore/seismo-dashboard](https://github.com/likeastore/seismo-dashboard) I've tried to use the model based on [GitHub personal tokens](https://help.github.com/articles/creating-an-access-token-for-command-line-use) instead of passwords, that simplifies server a bit allowing it to do not store and sessions, hashed passwords etc. If you interested, take a look at [likeastore/seismo/source/server.js](https://github.com/likeastore/seismo/blob/master/source/server.js#L40).

```js
'use strict';

angular.module('dashboardApp').controller('auth', function ($scope, $location, $cookieStore, authorization, api) {
	$scope.title = 'Likeastore. Analytics';

	$scope.login = function () {
		var credentials = {
			username: this.username,
			token: this.token
		};

		var success = function (data) {
			var token = data.token;

			api.init(token);

			$cookieStore.put('token', token);
			$location.path('/');
		};

		var error = function () {
			// TODO: apply user notification here..
		};

		authorization.login(credentials).success(success).error(error);
	};
});
```

As you can see, it just delegates the call to `authorizationn` service, which is very simple wrap of `$http`.

```js
'use strict';

angular.module('dashboardApp').factory('authorization', function ($http, config) {
	var url = config.analytics.url;

	return {
		login: function (credentials) {
			return $http.post(url + '/auth', credentials);
		}
	};
});

```

Ones server responds with success, controller will place the token to cookie.

The view is just a `form` with binded `ng-submit` event to call `auth.login()` function of controller.

```html
<div class="login-panel">
	<h1>Welcome to Analytics.</h1>
	<p>We are using GitHub for authorization. Please obtain your personal token and use it to sign in.</p>
	<form class="pure-form" ng-submit="login()">
		<input type="password" class="pure-input-1-3" placeholder="Personal Token..." name="token" ng-model="token" required/>

		<button type="submit" class="pure-button pure-button-primary">Sign in</button>
	</form>
</div>
```

## HTTP Interceptor

In case of any API call returns `401` we have to redirect user to login page. Angular's HTTP interceptor is great for that job. As you can see from `app.js` above, it's been pushed to pipe here:

```js
$httpProvider.responseInterceptors.push('httpInterceptor');
```

The interceptor implementation itself,

```js
'use strict';

angular.module('dashboardApp').factory('httpInterceptor', function httpInterceptor ($q, $window, $location) {
	return function (promise) {
		var success = function (response) {
			return response;
		};

		var error = function (response) {
			if (response.status === 401) {
				$location.url('/login');
			}

			return $q.reject(response);
		};

		return promise.then(success, error);
	};
});
```

## Placing Token to HTTP Headers

Finally, we need to supply that token as HTTP header parameter to all API calls that client issues. Again, you might notice in `app.js` there is an API initialization call.

```js
app.run(function (api) {
	api.init();
});
```

API initialization services looks like that,

```js
'use strict';

angular.module('dashboardApp').factory('api', function ($http, $cookies) {
	return {
		init: function (token) {
			$http.defaults.headers.common['X-Access-Token'] = token || $cookies.token;
		}
	};
});

```

This authentication model is very easy to integrate into any existing apps and just keep in mind while creating new ones.



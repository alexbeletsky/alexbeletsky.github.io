---
layout: post
title: "Testable API's with Node.js"
date: 2014-03-04 15:36
comments: true
categories: nodejs javascript tdd bdd nomocks
---

API is heart of modern web application. It's all about to make it easy to consume, scale and make sure it works as expected. Currently I follow "all open API methods must have tests" (AOAMMHT) principle. I used to work with .NET technologies, where testing of API's was about calling methods of corresponding controller object, which typically was unit testing - mocking up all controller dependencies, setting up expects of returned values.

I've changed my mind on testing with Node.js/Express.js development. For API's I prefer "end-to-end" testing: setting up user account, authentication, HTTP calls to server, real calls to DB and serving JSON payload back. API's have to be tested from consumer point of few to be able to give some meaningful results.

<!-- MORE -->

## Tools and Frameworks

Pretty standard setup: [mocha](http://visionmedia.github.io/mocha/), [chai](http://chaijs.com/), [request](https://github.com/mikeal/request).

Mocha is time proven tool for testing Node.js applications, Chai is good enough expectation framework and Request as one best HTTP clients I even worked with.

## Prepare application for testing

Dependencies above is installed via `npm install` and should be saved to `package.json` file (with `--save` option). Your project structure should have `test` folder inside, which mocha is using as default to look tests inside. Few files should be added there, current structure I have that works.

```plain
/test
	/specs
		auth.spec.js
		...
	common.js
	mocha.opts
	utils.js
	runMocha.js
```

`/api` folder is the one that would contain specifications for you API, `mocha.opts` - contains global mocha configuration, `common.js` is common require file, that all tests are using, `utils.js` - test helper that would contain everything you need during testing, `runMocha.js` utility that would be tests entry point.

### mocha.opts

```plain
--require ./test/common.js
--reporter spec
--ui bdd
--recursive
--colors
--timeout 60000
--slow 300
```

Mocha options allow to require some additional javascript file, as well as setting up global Mocha settings as what reporter to use, timeouts etc.

### common.js

```js
global.chai = require('chai');
global.expect = global.chai.expect;
```

To do not require `chai` in each spec files, it's possible to require it once and place to global scope.

### runMocha.js

```js
process.env.NODE_ENV = process.env.NODE_ENV || 'test';
process.env.TEST_ENV = process.env.TEST_ENV || 'test';

var exit = process.exit;
process.exit = function (code) {
	setTimeout(function () {
		exit();
	}, 200);
};

require('../source/server');
require('../node_modules/mocha/bin/_mocha');
```

The secret sauce is last 2 lines. Since `require` is synchronous we first "call" API server to get up and after that "call" mocha engine to start testing. So, inside each tests we can do real HTTP calls to real HTTP servers. No mocks.

### package.json

```js
// ...
"scripts": {
    "test": "node test/runMocha",
  },
// ...
```

Package file should contain script to call API tests with simple `npm test` command.

## Test driven API

Mocha is using BDD (behaviour driven development) approach to testing. Comparing to classical TDD, BDD encourage to write specifications in plain English, which works good especially when you just starting with particular feature.

What I typically do is just writing down specification of API, without any thinking of how to implement it. Give you example of something that I worked with recently.

```js
var request = require('request');
var testUtils = require('../utils');

describe('collections.spec.js', function () {
	describe('when non authorized', function () {
		it ('should not be authorized', function () {
		});
	});

	describe('when authorized', function () {
		describe('when new collection created', function () {
			describe('public', function () {
				it('should respond with 201 (created)', function () {
				});

				it('should create new collection', function () {
				});

				it('should have user', function () {
				});

				it('should collection be public', function () {
				});

				describe('and title is missing', function () {
					it('should respond with 412 (bad request)', function () {
					});
				});

				describe('with description', function () {
					it('should respond with 201 (created)', function () {
					});

					it('should create new collection', function () {
					});
				});
			});
		});

		// etc..
	});
});
```

This is kind of skeleton I to have before start anything else.

### Unauthorized access

If your API or part of it requires authorization, I prefer to test it.

```js
	var token, user, url, headers, response, results;

	beforeEach(function () {
		url = testUtils.getRootUrl() + '/api/collections';
	});

	describe('when non authorized', function () {
		beforeEach(function (done) {
			request.get({url: url, json: true}, function (err, resp, body) {
				response = resp;
				done(err);
			});
		});

		it ('should not be authorized', function () {
			expect(response.statusCode).to.equal(401);
		});
	});
```

`testUtils.getRootUrl()` returns qualified URL for API, depending on test environment. During development, it's just `http://localhost:3000` where your `server.js` started.

### Authorized access

Authorized access typically requires some kind of `access_token` sent either by headers or query string. Doesn't matter how, but `utils.js` must have method that would create new user and obtain access token from API. The actual implementation of such method would depend on your API auth mechanism.

All tests that required authorization access, should have such `beforeEach()`,

```js
beforeEach(function (done) {
	testUtils.createTestUserAndLoginToApi(function (err, createdUser, accessToken) {
		token = accessToken;
		user = createdUser;
		headers = {'X-Access-Token': accessToken};
		done(err);
	});
});
```

After `access_token` is acquired, it can be used as part of any authorized calls.

### Behavior tests

Now, everything is ready to test the behavior of API. Nothing fancy here, just act as clients do. Send HTTP requests, receive responses and check HTTP statuses. I'll just post some code, so it would give you direction.

```js
describe('when new collection created', function () {

	describe('public', function () {
		beforeEach(function () {
			collection = {title: 'This is test collection', public: true};
		});

		beforeEach(function (done) {
			request.post({url: url, headers: headers, body: collection, json: true}, function (err, resp, body) {
				response = resp;
				results = body;
				done(err);
			});
		});

		it('should respond with 201 (created)', function () {
			expect(response.statusCode).to.equal(201);
		});

		it('should create new collection', function () {
			expect(results.title).to.be.ok;
			expect(results._id).to.be.ok;
		});

		it('should have user', function () {
			expect(results.user).to.equal(user.email);
		});

		it('should collection be public', function () {
			expect(results.public).to.equal(true);
		});

		describe('and title is missing', function () {
			beforeEach(function (done) {
				request.post({url: url, headers: headers, body: {}, json: true}, function (err, resp, body) {
					response = resp;
					results = body;
					done(err);
				});
			});

			it('should respond with 412 (bad request)', function () {
				expect(response.statusCode).to.equal(412);
			});
		});

		describe('with description', function () {
			beforeEach(function () {
				collection = {title: 'This is test collection', description: 'description'};
			});

			beforeEach(function (done) {
				request.post({url: url, headers: headers, body: collection, json: true}, function (err, resp, body) {
					response = resp;
					results = body;
					done(err);
				});
			});

			it('should respond with 201 (created)', function () {
				expect(response.statusCode).to.equal(201);
			});

			it('should create new collection', function () {
				expect(results.description).to.equal('description');
			});
		});
	});
});
```

## Conclusions

I think dynamic languages as JavaScript is great for testing API's. Having no types eliminates "model-per-response" classes, `request.js` is great for making HTTP calls and `mocha` makes specifications output looks nice. So, nevertheless of back-end technology you can try to use the approach and see how it works for you.

The setup of tests and starting up of API services are so lightweight in Node.js that makes test-first API development nice and pleasant thing.
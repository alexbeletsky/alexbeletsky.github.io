---
layout: post
title: "Email, SMS and Push Notifications Server"
date: 2014-08-22 15:38
comments: true
categories: opensource nodejs javascript likeastore
---

I recently came up with very convenient component - [likeastore/notifier](https://github.com/likeastore/notifier). Notifier is easy to setup Email, SMS and Push Notifications Server. It's been open sourced few month, very specific to [Likeastore](https://likeastore.com) needs, but due to great [contributions](https://github.com/likeastore/notifier/graphs/contributors) it became very generic and can be used in you projects as well.

Once you need to setup infrastructure for notifications in your application, it will be really easy to do. It provides transport of [Mandrill](https://github.com/jimrubenstein/node-mandrill), [Twilio](https://github.com/twilio/twilio-node), [Android](https://github.com/ToothlessGear/node-gcm) and [Apple](https://github.com/argon/node-apn) push notifications.

<!-- MORE -->

## Starting up

The project already has comprehensive [README](https://github.com/likeastore/notifier/blob/master/README.md) that explains initial steps. At the moment, the server is available as github repository that you need to clone and prepare entry point for it. I'm thinking to pack the things into one `npm` module. If you see it makes sense, please let me know.

To start, you need to clone the repo,

```bash
$ git clone https://github.com/likeastore/notifier.git
```

And then create `app.js` file, there all setup of `notifer` taking place.

```js
var notifier = require('./source/notifier');

// TODO: configuration

// start the server
notifier.start(process.env.PORT || 7000);
```

Let's go straight to `configuration` part.

## Events, Actions and Executors

The `notifier` is HTTP server, that receives `event` and turns that event into corresponding `action`. One event could create as many actions as needed. Actions is something that later is going to be `executed`.

The basic setup consits of 3 parts: receive event, resolve event and execute it.

```js
notifier
    .receive('incoming-event', function (event, actions, callback) { /* ... */ })
    .resolve('created-action', function (action, actions, callback) { /* ... */ })
    .execute('created-action', function (action, transport, callback) { /* ... */ });
```

Say, we have `user-registered` event which we need to send a welcome email,

```js
notifier
	.receive('user-registered', function (e, actions, callback) {
		actions.create('send-welcome', {user: e.user}, callback);
	})
	.resolve('send-welcome', function (action, actions, callback) {
		db.users.findOne({email: action.user}, function (err, user) {
			if (err) {
				return callback(err);
			}

			if (!user) {
				return callback({message: 'user not found', email: action.email});
			}

			var data = {
				email: action.user,
				user: _.pick(user, userPick)
			};

			actions.resolved(action, data, callback);
		});
	})
	.execute('send-welcome', function (action, transport, callback) {
		var vars = [
			{name: 'USERID', content: action.data.user._id},
			{name: 'USER_NAME', content: action.data.user.displayName || action.data.user.name}
		];

		transport.mandrill('/messages/send-template', {
			template_name: 'welcome-email',
			template_content: [],
			message: {
				auto_html: null,
				to: [{email: action.data.email}],
				global_merge_vars: vars,
				preserve_recipients: false
			}
		}, callback);
	});
```

* `recieve` - receives event from outside and turns the event into corresponding action.
* `resolve` - step to extend action with additional data, email, name of user etc.
* `execute` - uses the transport to transmit the event

## Transports

Mandrill provides support from emails notifications,

```js
.execute('send-welcome', function (action, transport, callback) {
	var vars = [
		{name: 'USERID', content: action.data.user._id},
		{name: 'USER_NAME', content: action.data.user.displayName || action.data.user.name}
	];

	transport.mandrill('/messages/send-template', {
		template_name: 'welcome-email',
		template_content: [],
		message: {
			auto_html: null,
			to: [{email: action.data.email}],
			global_merge_vars: vars,
			preserve_recipients: false
		}
	}, callback);
});
```

Twilio is used from sms,

```js
.execute('send-verify-sms', function (a, transport, callback) {
		transport.twilio.messages.create({
			to: a.data.phone,
			from: '+12282201270',
			body: 'Verification code: 1111',
		}, callback);
	});
```

For Android push notifications,

```js
.execute('send-android-push-notification', function (a, transport, callback) {
	var tokens = [];
	tokens.push(a.data.token);

	transport.android.push({
		message: {key1: 'value1', key2: 'value2'},
		tokens: tokens,
		retries: 3
	}, callback);
});
```

For Apple push notifications,

```js
.execute('send-ios-push-notification', function (a, transport, callback) {
	var tokens = [];
	tokens.push(a.data.token);

	transport.ios.push({
		production: false, // use specific gateway based on 'production' property.
		passphrase: 'secretPhrase',
		alert: { "body" : "Your turn!", "action-loc-key" : "Play" , "launch-image" : "mysplash.png"},
		badge: 1,
		tokens: tokens
	}, callback);
});
```

## Configuration

Before the run, you need to make sure the `notifier` is configured properly. Security tokens and connections strings to MongoDB. If you use Logentries, you can provide token and all logs will be submitted there.

The `accessToken` is a shared secret between server and client, to prevent unauthorized access.

The [config](https://github.com/likeastore/notifier/blob/master/config/production.config.js) is using ENV variables on production and hardcoded values for development and staging.

## Clients

`notifier` can be used by any HTTP client, in most simple case `curl`,

```bash
$ echo '{"event": "incoming-event"}' | curl -H "Content-Type:application/json" -d @- http://notifier.likeastore.com/api/events?access_token=ACCESS_TOKEN
```

We already have Node.js client [DemocracyOS/notifier-client](https://github.com/DemocracyOS/notifier-client) and plan to have browser client as well. But in essence it's just HTTP post with `event` payload and `access_token` in query string, so basically it can be used from any language and platform.
---
layout: post
title: "Health Monitoring of Services and Databases"
date: 2014-09-19 18:27
comments: true
categories: nodejs monitoring http micro service
---

Once you are up and running your primary job is to monitor that all your services are in good health. If something bad happens, network issue or crashes - you have to be notified on that as soon as possible. Hardering of the HTTP services and databases became a one of primary activities for any startup.

There few products on market that can do that, [Pingdom](http://pingdom.com) and [New Relic](http://newrelic.com) probably the most know. But they are paid and not that flexible as I want. So, I've created [likeastore/heartbeat](https://github.com/likeastore/heartbeat) that for a month or so helps me to provide quality service.

<!-- MORE -->

It's really easy to setup and deploy on any server or Dokku/Heroku environment.

## Overview

The core of heartbeat is about ~300 [lines of code](https://github.com/likeastore/heartbeat/blob/master/source/hearbeat.js), so it's not a big deal to look inside to understand it principles of work. But in general, it's continuously running loop, that executes the small jobs who runs particular check strategy and report results if anything is wrong.

To make use of it you just need to clone the repo and update [config/index.js](https://github.com/likeastore/heartbeat/blob/master/config/index.js) with your configuration.

## Notifications

Heartbeat can use 2 types of notifications at the moment. [Mandril](https://mandrillapp.com/) based emails and [Twilio](https://www.twilio.com/) based SMSs.

Both requires you to create accounts there and provide API access keys.

That's what's configured on [transport](https://github.com/likeastore/heartbeat/blob/master/config/index.js#L67) section of configuration:

```js
transport: {
	mandrill: {
		token: 'fake-token'
	},

	twilio: {
		sid: 'fake-sid',
		token: 'fake-token'
	}
}
```

instead of `fake-token` and `fake-sid` you provide your real values there.

You have to specify the recipents for notifications, that happens in [notify](https://github.com/likeastore/heartbeat/blob/master/config/index.js#L56) section of configuration:

```js
notify: {
	email: {
		from: 'heartbeat@likeastore.com',
		to: ['devs@likeastore.com']
	},

	sms: {
		to: ['+3805551211', '+3805551212']
	}
}
```

As you can see, you specifying `from` and `to` emails, as well as `phone` numbers to send SMS.

## Monitoring

Now the main part, configure monitoring options. The [monitor](https://github.com/likeastore/heartbeat/blob/master/config/index.js#L10) section of configuration is responsible for that. At the moment, it provides provides 5 different monitoring strategies.

* HTTP
* JSON
* MongoDB
* Ping
* Resolve

### HTTP

The `HTTP` sends GET request to specified URL and checks the response code. If response code `!==200` the error notification is reported.

```js
http: [
    {
        url: 'https://likeastore.com'
    },
    {
        url: 'https://stage.likeastore.com'
    }
]
```

`http` is an array of urls to monitor.

### JSON

The `JSON` strategy suits the best for HTTP API's. It could request particular URL receive JSON response and compare it to expected result.

```js
json: [
    {
        url: 'https://app.likeastore.com/api/monitor',
        response: {
            "app":"app.likeastore.com",
            "env":"production",
            "version":"0.0.52",
            "apiUrl":"/api"
        }
    }
]
```

### MongoDB

MongoDB is a commonly part of web applications infrastructure. You need to have a connection string and then you specify the checking query. It could be `find` query or `insert` query.

```js
mongo: [
    {
        connection: 'mongodb://localhost:27017/likeastoredb',
        collections: ['users'],
        query: function (db, callback) {
            db.users.findOne({email: 'alexander.beletsky@gmail.com'}, callback);
        }
    }
]
```

### Resolve

Resolve is something that makes `heartbeat` a little different. It usually the thing that one `domain names` is configured for different `ip addresses`. As it name stands, `resolve` would resolve all ip's associated with domain name and ping them separately. If even one of them fails, the notification is created.

```js
resolve: [
	{
		name: 'google.com'
	}
]
```

`nslookup` for `google.com` returns,

```plain
› nslookup google.com
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	google.com
Address: 173.194.113.198
Name:	google.com
Address: 173.194.113.199
Name:	google.com
Address: 173.194.113.200
Name:	google.com
Address: 173.194.113.201
Name:	google.com
Address: 173.194.113.206
Name:	google.com
Address: 173.194.113.192
Name:	google.com
Address: 173.194.113.193
Name:	google.com
Address: 173.194.113.194
Name:	google.com
Address: 173.194.113.195
Name:	google.com
Address: 173.194.113.196
Name:	google.com
Address: 173.194.113.197
```

All that addresses will be checked separately.

### Ping

Finally it's `ping`, the simples operation. It just pings given `ip` address.


```js
ping: [
	{
		ip: '37.139.9.95'
	}
]
```

## Setting up the interval

You can specify the interval of checking. By default it's 10 seconds, but you can do more frequent checks if you want.

```js
interval: 5000
```

## Running and deploying

If the configuration is set, you can run heartbeat by,

```bash
$ node app.js
```

The log optionally could be forwared to [Logentries](logentries.com), by setting up [config/index.js#logentries](https://github.com/likeastore/heartbeat/blob/master/config/index.js#L6) section.
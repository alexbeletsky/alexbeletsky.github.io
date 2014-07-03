---
layout: post
title: "Using Mixpanel from Chrome Extensions"
date: 2014-05-22 11:15
comments: true
categories: javascript mixpanel analytics chrome
---

Recently we've released Google Chrome [extension](https://chrome.google.com/webstore/detail/likeastore/einhadilfmpdfmmjnnppomcccmlohjad?hl=en-US&utm_source=chrome-ntp-launcher) for [Likeastore](https://likeastore.com) that allows to extend your Google search results with relevant information from your likes. Development of Chrome extensions appeared to be interesting process, but essentially it's nothing more as web development, since primary technologies are still HTML/CSS/JS.

The most important thing for every startup are metrics. In web application we use [Mixpanel](https://mixpanel.com) together with [Google Analytics](https://www.google.com.ua/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CCkQFjAA&url=http%3A%2F%2Fwww.google.com%2Fanalytics%2F&ei=0sB9U_OyJMvc4QSYsYCoDA&usg=AFQjCNFz3Lrd3h9xlat60IUur_H8rmADdw&sig2=3VSXbRKyt9hgF6viPp2ahA&bvm=bv.67229260,d.bGE) and [Seismo](https://github.com/seismolabs). For browser extension I also needed some simple metrics, like "usage", "clicks", "shares" etc. Unfortunately, if you just try to reference `mixpanel-2.2.min.js` it won't work.

<!-- MORE -->

Instead, you will see error message in console,

```plain
Mixpanel error: 'mixpanel' object not initialized. Ensure you are using the latest version of the Mixpanel JS Library along with the snippet we provide.
```

There are few reasons for that. One is described in SO [question](http://stackoverflow.com/questions/15837450/content-security-policy-cannot-load-mixpanel-in-chrome-extension). But after I did same manipulations as answer mentioned it still didn't work. For extension I only have content script, there is no place there I can put `<script></script>` tag with markup. It seems like `mixpanel-2.2.min.js` have some dependencies that just prevents it to start normally inside extension, not usual web application.

I had no time to investigate the problem deeply, but instead found workaround that works.

## Mixpanel HTTP API

Besides platform-dependent libraries, Mixpanel provides access to [HTTP API](https://mixpanel.com/help/reference/http). This is the most "pure" way of communication between services, so I decided to try it.

Since, I already used `jQuery` in my application, I ended-up with such module,

```js
;(function (window) {
	var api = 'https://api.mixpanel.com';
	var app = window.app = window.app || {};

	var mixpanel = {};

	function init(token, user) {
		mixpanel.token = token;
		mixpanel.distinct_id = user.email;
	}

	function track(event) {
		var payload = {
			event: event,
			properties: {
				distinct_id: mixpanel.distinct_id,
				token: mixpanel.token,
				browser: app.browser.name
			}
		};

		var data = window.base64(JSON.stringify(payload));
		var url = api + '/track?data=' + data;

		$.get(url);
	}

	app.analytics = {
		init: init,
		track: track
	};

})(window);
```

Here, I'm using [this](https://gist.github.com/alexanderbeletsky/3f850467bc23d40ef51e) implementation of `base64` (copied from `mixpanel-2.2.js`). So, to track the `event` you just HTTP GET to Mixpanel API, with a payload of base64'ed JSON string of `event` and `properties`.

## Initialization

To init `mixpanel` you need both `token` and `distinct_id`. The `distinct_id` is unique value, that Mixpanel generates itself and stores the value to cookie, to identify the user. If you send events from different applications, say web app, mobile web extension etc. - I would recommend to user `mixpanel.alias()` call, so you "link" unknown `distinct_id` with some user's attribute (`id` from database or `email` would work).

On our web application, once user logged on we call `mixpanel.alias()` with email of user, so all other applications could use that as `distinct_id` of user.

The entry point of content script is `run()` method, where user is fetched from server and initialize `mixpanel` module,

```js
var ready = function (user) {
	app.analytics.init('[MIXPANEL_TOKEN_HERE]', user);
};

return {
	run: function () {
		$.get(api + '/users/me')
			.done(ready)
			.done(search)
			.fail(login);
	}
};
```

## Tracking

Once content script injects it's HTML into target page DOM, it's possible to listen to DOM events and call `.track()` function.

```js
self.bindEvents = function () {
	self.$.find('.item a').click(function () {
		app.analytics.track('search extension results clicked');
	});

	self.$.find('.ls-more a').click(function () {
		app.analytics.track('search extension more clicked');
	});

	app.analytics.track('search extension');
};
```

## Conclusions

I consider Mixpanel HTTP API as very nice substitution of library, in those rare cases when library is not applicable. This would work perfectly if you need simple events tracking. If you interested, feel free to check [sources](https://github.com/likeastore/browser-extension).
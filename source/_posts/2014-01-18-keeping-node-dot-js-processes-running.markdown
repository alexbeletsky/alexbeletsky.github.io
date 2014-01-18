---
layout: post
title: "Keeping Node.js Processes Running"
date: 2014-01-18 13:38
comments: true
categories: nodejs javascript
---

Node.js/Express.js is great for Web API's and applications. In contrast to known enterprise technologies, Node.js is very special. It's single process/threaded environment. In case of unhanded exception occurred Node.js virtual machine simply stops, leaving application in unresponsive state.

Due to `async` nature of Node.js `try/catch` not always works, even with `domains` and stuff you have a chance that application crashed on production while you sleep.

<!-- More -->

To mitigate the issue few [known solutions](http://stackoverflow.com/questions/1972242/auto-reload-of-files-in-node-js) exist, common idea is that there is watchdog that keeping eye on `node` process and if crashed, restarts application again.

Recently I've used great library by [@mafintosh](https://github.com/mafintosh) called [respawn](https://github.com/mafintosh/respawn). I liked it's minimalistic style and decided to try it out.

The bare-bones code is very simple. Without modification of your application, just create file `monitor.js` with nearly such code:

```js
var respawn = require('respawn');

var monitor = respawn(['node', 'server.js'], {
    env: {ENV_VAR:'test'}, // set env vars
    cwd: '.',              // set cwd
    maxRestarts:10,        // how many restarts are allowed within 60s
    sleep:1000,            // time to sleep between restarts
});

monitor.start(); // spawn and watch
```

`monitor` will spawn new node process and in case of crash it will be restarted. You can also specify `maxRestars` (I recommend to do that, if something is really bad it won't be restarted infinitely) and `sleep` time.

I've tried that, by implementing `/fail` end-point in my app, to see that `respawn` really works.

```js
app.get('/fail', function (req, res, next) {
	setTimeout(function () {
		var nu = null;
		nu.access();

		res.send('Hello World');
	}, 1000);
});
```

if I try to hit `/fail` I'll see no results in browser, but if I go back to `/` the application is running in normal state.

But simple respawning of application is not complete solution. You need to know what exactly happened to be able to fix issue. [Proper logging](http://beletsky.net/2013/07/think-ahead-think-logging.html) of your application is essential. I'll show my small setup around `respawn` that send critical message to [Logentries](https://logentries.com), so all crashes are logged.

```js
var respawn = require('respawn');
var util = require('util');
var logger = require('./source/utils/logger');

var proc = respawn(['node', 'app.js'], {
	cwd: '.',
	maxRestarts: 10,
	sleep: 1000,
});

proc.on('spawn', function () {
	util.print('application monitor started...');
});

proc.on('exit', function (code, signal) {
	logger.fatal({msg: 'process exited, code: ' + code + ' signal: ' + signal});
});

proc.on('stdout', function (data) {
	util.print(data.toString());
});

proc.on('stderr', function (data) {
	logger.error({msg: 'process error', data: data.toString()});
});

proc.start();
```

(details of logger you can find in this [post](http://beletsky.net/2013/07/think-ahead-think-logging.html))

All process output is goes to `stdout`, which is convinient for development, but in case of `stderr` or `exit` everything is logged to cloud and notification to `dev-team` sent.

It worked really nice, now I'm not worry even if something bad happens on production, `respawn` will make sure that rest of users are not affected. As a developer you can much quicker found bug and push hotfix.
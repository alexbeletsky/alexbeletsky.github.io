---
layout: post
title: "NPM for Everything"
description: NPM as dependency manager, task runner both for front and back end.
date: 2015-04-02 15:26
comments: true
categories: npm javascript frontend backend
facebook:
 image: https://lh3.googleusercontent.com/-dUeSHlXoO0g/VR1NxzODdII/AAAAAAAAlNc/YwW8j0F2EuA/w2234-h1160-no/Screenshot%2B2015-04-02%2B16.png
twitter_card:
 image: https://lh3.googleusercontent.com/-dUeSHlXoO0g/VR1NxzODdII/AAAAAAAAlNc/YwW8j0F2EuA/w2234-h1160-no/Screenshot%2B2015-04-02%2B16.png
---

JS ecosystem is famous for it's wide range of packages and tools. Besides the default package manager, which comes with Node.js - NPM (node package manager), there are Bower.js, Jam.js and many more.

For a quite long period, my default setup was like - NPM for all server-side components and Bower.js for client-side stuff. It worked quite good together, especially in conjunction with Require.js.

But recently, I switched to *NPM only* setup. That allowed me to use CommonJS style both front and back ends and got rid of task runners as Grunt and Gulp.

<!-- MORE -->

# CommonJS

Doing a lot of back-end coding I really get used to `CommonJS` style, which is simple, practical and powerful. CommonJS modularization is plain as,

```js
var module = {
};

// export the module
module.exports = module;
```

and

```js
// use the module
var module = require('./module');
```

Node.js highly popularized Common.js approach. And NPM appeared to be one of the best package managers ever developed.

But for front-end, NPM was not that suitable. First of all, because of browsers do not understand Common.js style of modules. Second, `npm` modules are being placed to `./node_modules` folder, with is typically the root of project, that made things complicated to refer those scripts from `index.html` and configure server to server static resources from `./node_modules` folder.

Browserify came to change this.

# Browserify

Browserify is JavaScript transpiling tool, with allows to:

1. Use Common.js style for front-end applications.
2. Allowed to use various `npm` packages directly in a browser.

A lot of pure front-end frameworks and libraries were published to `npm`. In fact, it's more and more used for front-end libraries as [according to](http://blog.npmjs.org/post/101775448305/npm-and-front-end-packaging) NPM developers.

Of cause, nothing comes for free. The cost of Browserify is a *build step*. You have to introduce to transform Common.js application into a *bundle* which is loaded and executed by a browser.

Moreover, some packages are still being hosted on Bower. Even, it's possible to use Bower packages together with NPM packages (by means of debowerify transformation) I quickly realized, that this approach is very unpractical.

So, during the last few months my main [contributions](https://github.com/pulls?q=is%3Apr+author%3Aalexbeletsky+is%3Aclosed) on GitHub was adopting of some front-end packages, to be conformant to Common.js and publishing them on NPM after.

# The setup

The NPM now is the only one package manager to use, both for front-end and backend.

All dependencies are mentioned on one file `package.json` instead of several files.. and you will never forget to run `bower install`, since `npm install` will do install everything.

```js
  "dependencies": {
    "async": "^0.9.0",
    "babelify": "^5.0.4",
    "body-parser": "^1.9.0",
    "colors": "^1.0.2",
    "cors": "^2.4.2",
    "envify": "^3.4.0",
    "express": "^4.9.5",
    "method-override": "^2.2.0",
    "moment": "^2.8.3",
    "morgan": "^1.3.2",
    "node-logentries": "^0.1.4",
    "react": "^0.13.1",
    "respawn": "^1.0.1"
  }
```

The server side code,

```js
var express = require('express');

// configurations...

app.listen(port, function () {
    logger.info('editor-app listening on port ' + port + ' ' + env);
});
```

The client side code,

```js
var React = require('react');

var Component = React.create({

});

module.exports  = Component;
```

The same style, feels nice and clean.

# No task runners

NPM is not only package manager. It's also very simple task runner (or scripts runner). There is a special section, called `scripts` responsible for that. Originally it's been used to run some package related tasks (run tests, perform pre/post install configurations).

Traditionally Grunt.js and Gulp.js are ones that used dealing with compiling static assests. But CLI interface of many tools and stream piping could replace them.

As example, I'll show pretty typical setup:

* build JavaScript with `browserify` and apply minification
* build Sass with `node-sass` and apply minification
* run watcher and rebuild changed assets

## Building JavaScript

The `browserify` need to be installed globally,

```bash
$ npm install -g browserify
```

In `package.json`, in `scripts` section, will add such script,

```js
  "scripts": {
    "build-js": "browserify public/js/app.js -o public/build/app.js",
```

Since I also use several transformations in my applications, the Browserify have to properly configured for that. It check's if `package.json` contains `browserify` section, and if it does - will use it contents as own options.

```js
  "browserify": {
    "transform": [
      "babelify",
      "envify"
    ]
  },
```

## Building Sass

As with Browserify, `node-sass` need to be installed globally,

```bash
$ npm install -g node-sass
```

Then `package.json` changes,

```js
  "scripts": {
    "build-js": "browserify public/js/app.js -o public/build/app.js",
    "build-sass": "node-sass public/sass/main.scss public/build/main.css",
```

## Minification

Now, let's minify both assets using one of the best concepts ever - Unix pipes.

We need two packages additionally, `uglifyjs` and `cleancss`.

```bash
$ npm install -g uglifyjs clean-css
```
And corresponding scripts,

```js
  "scripts": {
    "build-js": "browserify public/js/app.js -o public/build/app.js",
    "build-sass": "node-sass public/sass/main.scss public/build/main.css",
    // minifying
    "build-min-js": "browserify public/js/app.js | uglifyjs -o public/build/app.min.js",
    "build-min-sass": "node-sass public/sass/main.scss | cleancss -o public/build/main.min.css",
```

## Watching

Now, let's apply watching. One of the watchers I use for several years already, that works best for me is `nodemon`.

`nodemon` is not only able to watch file changes and restart node processes if necessary, but also running custom commands.

```js
  "scripts": {
    "build-js": "browserify public/js/app.js -o public/build/app.js",
    "build-sass": "node-sass public/sass/main.scss public/build/main.css",
    "build-min-js": "browserify public/js/app.js | uglifyjs -o public/build/app.min.js",
    "build-min-sass": "node-sass public/sass/main.scss | cleancss -o public/build/main.min.css",
    // watching
    "watch-js": "nodemon -e js -w public/js -x 'npm run build-js'",
    "watch-sass": "nodemon -e scss -w public/sass -x 'npm run build-sass'",
```

## Complex tasks

Typically, you need to run few tasks simultaneously, e.g. build all scripts or watch all application. We can combine commands, by running background tasks,

```js
    "build": "npm run build-js & npm run build-sass",
    "watch": "npm run watch-js & npm run watch-sass",
```

To build full app,

```bash
$ npm run build
```

or to *watch* it,

```bash
$ npm run watch
```

## Using dev-dependencies

Global packages are not cool, cause they might be missing on developers box there application is cloned. Moreover, dependency management becomes a bit more complex, if you need to work on different projects depending on different tools versions. As [@bcomnes](https://github.com/bcomnes) very correctly noticed, all tools as `browserify`, `uglify` etc. can be installed as local `dev-dependency`, which are fetched during `npm install` and will be available to call inside project folder as global commands,

```bash
$ npm install --save-dev browserify uglifyjs node-sass clean-css
```

So, the `package.json` got such update,

```json
"devDependencies": {
  "browserify": "^9.0.7",
  "clean-css": "^3.1.9",
  "node-sass": "^2.1.1",
  "uglifyjs": "^2.4.10"
},

```

# Takeaways

* Once you implement front end library, consider [UMD](https://github.com/umdjs/umd) so your modules are reusable in any environment.
* Embrace CommonJS, use Browserify/Webpack/Whatever to use CommonJS (or ES6) on client.
* For many cases, you really don't need Grunt or Gulp, all is easily doable with NPM scripts.

As example, please refer to [brick](https://github.com/alexbeletsky/brick), the project I currently use as boilerplate for new applications.

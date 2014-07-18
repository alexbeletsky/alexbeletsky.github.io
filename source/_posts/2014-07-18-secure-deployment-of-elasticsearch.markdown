---
layout: post
title: "Secure Deployment of ElasticSearch"
date: 2014-07-18 12:34
categories: elastic nodejs security docker digitalocean
---

Remember my success [story](http://beletsky.net/2014/05/got-tired-of-mongodb-full-text.html) of moving from MongoDB full text search index to [ElasticSearch](http://www.elasticsearch.com)? After about one month of work our search service unexpectedly stopped. I received a bunch of emails from [Logentries](http://logentiries.com) and [NewRelic](http://newrelic.com) that server is no longer responsive.

Once I logged on to [DigitalOcean](https://www.digitalocean.com/?refcode=de56d081b272) account, I've seen message from administrators that server is sending a lot of traffic (UDP Flood), very likely compromised and therefore stopped. The link they give as [instructions](https://www.digitalocean.com/community/questions/my-droplet-is-locked-by-support-staff-because-because-of-an-outgoing-flood-or-ddos-what-do-i-do) to fix the problem was quite comprehensive, but the most important info was in comments. A lot of people who were hit by problem had ElasticSearch deployed on their machines.

<!-- MORE -->

It appeared, if ES is left on default configuration with port opened outside, it will be easy target for bad guys, both of Java and ES vulnerability. Basically, I had to restore server from scratch, but this time I'm not goint to be naive, server have to be properly secured.

I'll describe my setup that involves: Node.js, Dokku / Docker, SSL.

## Clean DigitalOcean image

I had to reinstall my machine from scratch, so instead of creating plain Ubuntu 14 server, but I decided to go dokku/docker path. Something that I tried [before](http://beletsky.net/2013/09/paas-in-your-pocket-with-dokku.html) and very happy with the [results](http://beletsky.net/2013/08/digitalocean-plus-dokku-equals-10-heroku.html). [DigitalOcean](https://www.digitalocean.com/?refcode=de56d081b272) offers pre-packed image with dokku/docker already on board. As usually it takes just a few seconds to spin up machine on DO.

The plan was the following: deploy ElasticSearch instance inside Docker container, disable dynamic script features of Elastic, deploy Node.js based proxy server with custom authentication by Dokku and link those containers, so only Node.js proxy will have access to Elastic. Finally, all traffic between will by crypted by SSL.

The benefits are obvious, no more `:9200` is outside the machine, one potential vulnerability with dynamic script feature is disabled, only authenticated client could access Elastic server.

## Deployment of ElasticSearch in Docker

First we need to have Docker image of ElasticSearch. There few ES plugging for Dokku, I decided to install one by myself, since I thought it is easier to configure. There is a great instruction from Docker [here](https://registry.hub.docker.com/u/dockerfile/elasticsearch/).

```bash
$ docker pull docker pull dockerfile/elasticsearch
```

Once the image is there, we need to prepare volume there Elastic stores data and configuration.

```bash
$ cd /
$ mkdir elastic
```

Elastic folder should contain `elasticsearch.yml` file, with all required configurations. My case is very simple, I have a cluster of one machine, so default configuration applies to me. One thing, that I mentioned above - I need to disable dynamic scripting features.

```bash
$ nano elasticsearch.yml
```

The content of the file is just one string,

```plain
script.disable_dynamic: true
```

Once it's done, we are ready to launch the server inside Docker container. Since, it could be done few times (during configuration and debugging), I've created a small shell script,

```bash
docker run --name elastic -d -p 127.0.0.1:9200:9200 -p 127.0.0.1:9300:9300 -v /elastic:/data dockerfile/elasticsearch /elasticsearch/bin/elasticsearch -Des.config=/data/elasticsearch.yml
```

Please note the important thing, `-p 127.0.0.1:9200:9200` - here we are binding `:9200` to be only accessible on `localhost`. I've spend some hours to close `:9200` by `iptables` without any success, but that thing works as expected. Thanks a lot to [@darkproger](https://twitter.com/darkproger) and [@kkdoo](https://twitter.com/kkdoo) for great help.

`-v /elastic:/data` will map the containers volume `/data` to local one `/elastic`.

## Front-end Node.js server

Now, we need to deploy front-end proxy server. It would proxy all traffic from `http://localhost:9200` into outside world, securely. I've created small project, based on [http-proxy](https://github.com/nodejitsu/node-http-proxy) called [elastic-proxy](https://github.com/likeastore/elastic-proxy). It's very simple one and can be easily re-used.

```bash
$ git clone https://github.com/likeastore/elastic-proxy
$ cd elastic-proxy
```

The server itself,

```js
var http = require('http');
var httpProxy = require('http-proxy');
var url = require('url');

var config = require('./config');
var logger = require('./source/utils/logger');

var port = process.env.PORT || 3010;
var proxy = httpProxy.createProxyServer();

var parseAccessToken = function (req) {
	var request = url.parse(req.url, true).query;
	var referer = url.parse(req.headers.referer || '', true).query;

	return request.access_token || referer.access_token;
};

var server = http.createServer(function (req, res) {
	var accessToken = parseAccessToken(req);

	logger.info('request: ' + req.url + ' accessToken: ' + accessToken + ' referer: ' + req.headers.referer);

	if (!accessToken || accessToken !== config.accessToken) {
		res.statusCode = 401;
		return res.end('Missing access_token query parameter');
	}

	proxy.web(req, res, {target: config.target, rejectUnauthorized: false});
});

server.listen(port, function () {
	logger.info('Likeastore Elastic-Proxy started at: ' + port);
});
```

It proxies all the requests and only by-pass ones, who provide `access_token` as query parameter. The `access_token` value is [configured](https://github.com/likeastore/elastic-proxy/blob/master/config/production.config.js) on server, by applications environment variable `PROXY_ACCESS_TOKEN`.

As you already [prepared](https://www.digitalocean.com/community/tutorials/how-to-use-the-dokku-one-click-digitalocean-image-to-deploy-a-python-flask-app) Dokku all you need to do is to push application to your server.

```bash
$ git push master production
```

After the deployment, you should go to server to configure applications environment,

```bash
$ dokku config proxy set PROXY_ACCESS_TOKEN="your_secret_value"
```

I wanted to have SSL and it's very easy to configure with Dokku. Just place your `server.crt` and `server.key` to `/home/dokku/proxy/tls` folder.

Proxy have to be [redeployed](https://github.com/scottatron/dokku-rebuild) afterward, to apply configuration changes. Try proxy by hitting it's URL, in my case [https://search.likeastore.com](https://search.likeastore.com). If everything is good, it will reply,

```plain
Missing access_token query parameter
```

## Link containers

We need to link both containers, so Node.js application is able to access ElasticSeach container. I really liked [dokku-link](https://github.com/rlaneve/dokku-link) plugin, that does exactly that's required in very easy way. So, will install it

```bash
$ cd /var/lib/dokku/plugins
$ git clone https://github.com/rlaneve/dokku-link
```

Now, we need to link containers,

```bash
$ dokku link proxy elastic
```

And application have to be redeployed again. If everything is good, you will be able to access the server by `https://proxy.yourserver.com?access_token=your_secret_value` and see the response from ElasticSearch,

```js
{
	status: 200,
	name: "Tundra",
	version: {
		number: "1.2.1",
		build_hash: "6c95b759f9e7ef0f8e17f77d850da43ce8a4b364",
		build_timestamp: "2014-06-03T15:02:52Z",
		build_snapshot: false,
		lucene_version: "4.8"
	},
	tagline: "You Know, for Search"
}
```

## Reconfigure the client

Depending on a client you use, you need to apply a little tweak to configuration. Basically, we need to use `access_token` for all our request. For Node.js applications,

```js
var client = elasticsearch.Client({
	host: {
		protocol: 'https',
		host: 'search.likeastore.com',
		port: 443,
		query: {
			access_token: process.env.ELASTIC_ACCESS_TOKEN
		}
	},
	requestTimeout: 5000
});
```

Again, `ELASTIC_ACCESS_TOKEN` is env variable to hold `your_secret_value` token.

Now, restart the application, make sure everything is running and exhale.

**PS**. I would like to say thank to [DigitalOcean](https://www.digitalocean.com/?refcode=de56d081b272) for support and a little credit I received as downtime compensation. Thats awesome.
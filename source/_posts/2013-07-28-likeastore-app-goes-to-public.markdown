---
layout: post
title: "Likeastore App Goes to Public"
date: 2013-07-28 09:30
comments: true
categories: likeastore applications
---

About 4 month the project [started](http://beletsky.net/2013/03/likeastore-application-built-on.html) on hackathon and called [Likeastore](https://likeastore.com/). Yesterday, it's been pushed out to public and I feel really great about that. Started out as quick hack, it eventually became a real product.

Bootstrapping is always hard, despite of the actual product size. Finishing something that other people will see, just doubles the stake - you could not fail, wish to do best of the best. It's been really breathtaking journey (or I would call it beginning of the path), so if you are interested I'll share some details of product and it's development.

<!--More-->

![likeastore](/images/blog/likeastore-home.png)

## Likeastore in few simple words

We all connected to a different social systems, like twitter, github or stackoverlow. All of those are constant flow of information and we clasiffy that information by "liking" them.

The usual problem though, is it's very hard to remember where you saw some information. For instance, I just remember *"hm..I read great post of configuring nginx for Node.js application.."*, but totally forget - was on github readme, twitted blog post or answer on stackoverflow.

>Likeastore tool that helps you to survive information overload.

Likeastore fixes the problem, it stores all information that you might be interested in, by collecting your "likes" throught a different social applications and allow you to search that information.

## After hackathon time

We took a second place on hackathon and a lot of people were interested in Likeastore idea. We just wanted to make it right and enhance the code we already had.

Our plan was simple: make everything we had right, improve the UI and found reliable hosting for our product. As this is done, we go for a private mode, sharing application with limited number of users. If everything is good, will release it public. Pretty standard approach for different kinds of SaaS.

At May, 2013 we had our application deployed on [AppFog](https://www.appfog.com/) and subscribers been invited.

## Private beta results

After few hours on production site running, we've got first users, first real data and even first feedback in twitter.
But what's more important, we've got a logs full of errors and understanding we are not that far from previous milestone.

Application stopped to work on next day. We've [moved](http://beletsky.net/2013/07/why-we-moved-from-appfog-to-nodejitsu.html) to another PaaS [Nodejitsu](https://www.nodejitsu.com/). A lot of different things happen.

Looking back, I have to say - *it's great we didn't show it to public immediately*, cause it would be a bit shameful. Decrypting the logs, gave us better understanding of problems you face collecting big amount of data through different API's. There was obviously huge amount of work to make it better.

## Focusing on quality

Originally, we've planned to push more features to public version: facebook and tumlr connectors, full text search and other stuff. But private beta clearly showed - the *focus have to be on quality*.

We literally re-write application from scratch, both server side and client. Applied integration tests and unit tests whereever it's possible, also covering main application pathes with zombie.js [end-to-end](http://pixelhunter.me/post/54753803233/end-to-end-testing-with-zombie-js-mocha-js-and) tests.

Our [API](https://github.com/likeastore/app/tree/master/source/api) became stable and fully tested, our [client](https://github.com/likeastore/app/tree/master/public/js) solid rock and very reliable. The trickiest part of application, the [collector](https://github.com/likeastore/collector) - it became smarter, with a scheduler that schedules optimal requests to avoid API rate limits.

## Make the product beautiful

Beautiful UI/UX for applications like a crucial. And that's why I'm happy that [@voronianski](https://twitter.com/voronianski) is a part of team.

He did amazing job by visualizing both site and application. Design became very clean, contrast and looks great on numerous of devices, from smart phones to huge desktop monitors.

Following the modern trend of flat design, we've picked up colors and styles, showing the way we see application, as modern and useful. Just *making the things look nicer* would be a real further strategy for us.

## Security

We collect users private data, it was obvioulsy just impossible to give a chance of someone else access it. So, have to secure the application with SSL connection, so all trafic between API and browser is crypted. SSL became a quite hard, first because certificate is rather expensive, second you have to install on server, which I never did before.

The importance of SSL left us with a no-go on [Nodejitsu](https://www.nodejitsu.com/). Nodejitsu policies allows to have SSL connections only on "Bussiness Plan", which costs 120USD/month. That was not that money we ready to put on table. We had to have change the way we host application. Basically, it meant - move out from PaaS to IaaS, and do all the stuff on your own.

I personally felt bad about, since we've really get used to PaaS. But, we've found a way around, having place own application on [Digital Ocean](https://www.digitalocean.com/) cloud, in conjunction with [dokku](https://github.com/progrium/dokku), which allows us to Heroku-like deployment model. That also gave me the chance to play vagrant/linux/docker more that I could ever image. I contributed [dokku](https://github.com/progrium/dokku) some important stuff as SSL and ENV support and that gave ability to use it for production needs.

## Current technology stack

So, a little bit of geeky info. We running on:

* [Digital Ocean](https://www.digitalocean.com/), 2 droplets one for staging and one for production
* [Nginx](http://nginx.org/) 1.4, ssl, routing and balancing 2 Node.js servers on each droplet
* [Node.js](http://nodejs.org/) 0.8.25 + [Express.js](http://expressjs.com/) based API
* [MongoHQ](http://www.mongohq.com/home) deplyment for staging and production database
* Each Node.js server runs in [docker](http://www.docker.io/) container
* Deployments and docker orchestration by [dokku](https://github.com/progrium/dokku)
* [AngularJS](http://angularjs.org/) driven client
* Handcrafted CSS, HTML (no frameworks)

Each of these is very interesting and demands dedicated posts. Hope, we do some in future.

## Conclusions

Just current results: 18 hours after, we have 40 users, 62 network connections, 10861 item collected. That's the motivating stuff, so we feel very positive to carry that on.
---
layout: post
title: "First Week in The Air"
date: 2013-08-05 05:40
comments: true
categories: applications likeastore
---

It's been a week how [Likeastore](https://likeastore.com/) application went public. I was amazed with initial reaction and feedback. Right now, we have 730 sign-ups, 844 social networks connected and 117950 likes collected. That mean, we had ~100 signups per day and it's growing.

Here is a little retrospective on the things happened through that seven days.

<!-- More -->

## Few hours after release

We've deployed about 6.00 PM at Saturday, sending out a bunch of invitation latters. Right after that, users began to register.

Small script that sends *"New user registered"* notification has been rolled out as well, so it was easy to see how things are going. Saturday both me and [@voronianski](https://twitter.com/voronianski) were completely tired. No doubt, it has been quite an exhausting journey, we worked hard last 3 weeks at nights and weekends, so I just came home and fall asleep, just checking that application is still running.

## Unpredictable thing

Next morning, I found a bunch of emails in my box, maybe 40-50 subscribed. Mostly from my "nearest" network, after notification in twitter and facebook.

But then, something unpredictable happened. [@voronianski](https://twitter.com/voronianski) skyped me that we are about to be published at amazing [Collective #74](http://t.co/4DinPVlcdk) issue.

That gave us a lot of traffic and sign-ups. Monday and Tuesday, we had a rush hours then every minute new user sign-up. My phone was ringing all the time, and my email box was full of notifications. People were [tweeting](https://twitter.com/search?q=likeastore&src=typd) about, we recieved few request for new service connectors as well.

I have to say, I was really pleased with that. We did something noticeable, something that other people liked.

## First problems

Real user experience shows real application issues. We had few..

First is that our app failed to work in IE. It appeared that our *nginx* configuration. Since we are using [Dokku]() for deployment, it have to be fixed there. Thanks to Dokku community, the issue has been resolved rather quickly and fix being [pushed](https://github.com/progrium/dokku/commit/33a3b85674e92fe883ba3151dee861f53914718a).

Another thing, that was not possible to see without some *real* data - stackoverflow OAuth token is very short living.. So, if you enabled stackoverlow connector, next day it fail to collect the data, since token in not valid any more. Unfortunately, there is no way to workaround this issue automatically, only ask user to re-enable the network.

We had several outages of data collection caused be bugs not noticed on staging. Also, to improve *initial collecting velocity*, we run 2 instances of collector in parallel. That worked, so just subscribed users are served fast, but it's not ideal yet.

## What's coming?

The last week was really motivating. We are continue to improve the service and expecting new users to come.

Through the next month we'll be focusing on:

* **Facebook connector** - the top most requested connector to be implemented.
* **Likes indexing** - currenly implemented search is very simple, just to show the idea, what is really needed is to perform the search based in likes content.
* **UI improvements** - we are still improving the design of application, to make it attractive and unique.
* **Performance** - our core component [collector](https://github.com/likeastore/collector) will be deployed on dedicated core 2 machine and re-written to be easily scaled.
* **CDN** - currently all static resources are handled by node.js, which is terrible slow, will move all our stuff to Amazon CloudFront.
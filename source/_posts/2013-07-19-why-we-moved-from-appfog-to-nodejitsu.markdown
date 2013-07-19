---
layout: post
title: "Why We Moved From AppFog to Nodejitsu"
date: 2013-07-19 19:36
comments: true
categories: likeastore nodejs paas
---

[Likeastore](http://likeastore.com/) started to use [AppFog](https://www.appfog.com/) as PaaS during private beta campaign. That was great idea initially since AppFog offers really nice conditions: 8 running instances with 256MB or RAM, custom domain names, support and all relative services. I have to say, it did work fine at the beginning, allowing us to push product forward and show it to our subscribers.

But very fast we realized that AppFog does not suite us at lot.

<!--More-->

## Why to mess up with PaaS?

If you never tried things like [Heroku](https://www.heroku.com/), [AppFog](https://www.appfog.com/), or [AppHabor](https://appharbor.com/) you might have a question, why to pay for something you "almost" can do your self just by renting VPS.

Renting VPS seems to be nice idea, but the problem is - you taking to much responsibility to manage the server by your own. You have to have certain skills to configure nginx, git repositories, ssh keys etc. It's possible to do, but it takes a lot time.. time you can spend on code something will be spend to configure something.

Another very important point is deployments. Usually deployments are hard and time consuming, since you have to do them manually. But with PaaS all you have to do to deploy the up is either, `git push` to some remote repo.. or call a special script like `af push app`, to do all the magic.

For small companies and side-projects PaaS is a really good opportunity to actually ship something, instead of fail to customize the web server.

## Why AppFog?

I've heard about AppFog before from twitter and hacker news. It looked very attractive.

But it was not the only one on the market. So, really competitive feature for me was: 8 running instances, for 20 USD. Very good price. We need 4 at that time, so AppFog was very good choice.

But as always, with time you start to see some negative moments. Something that lead us to finally drop AppFog. I would like to quickly go through good and less good things.

## Dashboard and UI

AppFog dashboard is quite nicely designed. But sometimes I felt difficulties to just find the things.

I was not happy with performance of dashboard as well. It was really slow for me usually, so you click to check instance state and wait for 30 seconds while the page opens. You can leave with that, but the time comes than you start to hate it.

## AppFog CLI

To create / run / stop applications on AppFog, you get a special command line utility, called `af`. It's typically for all PaaS to have some CLI to communicate to it. It's easy to install and configure, everything is nice.

It's written on Ruby, so you won't have any problems on Mac, but Ruby have to installed on Windows machine.

Once it's installed you have to login to you AppFog account and start to deploy.

## Deployment experience

Even if CLI itself was quite nicely done, deployment experience was not so nice - it was too slow.

Node.js is deployment-friendly platform. No build steps, no linking, no packaging - nothing. All you have to do, is to push the sources to remote machine. Sure, PaaS is doing a lot magic behind the scences, like firing new virtual machines and configuring network interfaces etc.

And all that magic for AppFog works too long.

Deployment could took up to 40 seconds, which is fine if you do it once in week. But it bothers you a lot while you do that once in hour.

## Release management

If your plan is continuous delivery and frequent deployments, you have to be ready for frequent rollbacks as well. Without that feature, you are in trouble.

AppFog does not afford anything like that. Correctly say, you have to do it manually, like tagging sources, then fetching by tag and push again.

That's not big issue, I would say. Nevertheless, developers are too optimistic thinking that everything works great (and indeed in always works fine, on my machine), but the time you see issues on production you just panic and loosing control of what to do.

## Support

Even if you running tiny bussiness and no one hurt if your application is down, still you expect that you'll get help if you are in trouble. That's the whole idea of support, especially if you pay money for it.

AppFog support is not good. Few times my tickets were unanswered for 2 days. Then they answered I found the reason or was able to fix it by myself.

This is just makes very bad impression of service. You start to think, it's not so reliable, so the time you'll be really screwed, they won't come help.

## Load balancing and cookies

So, all mentioned above is something that we would prioritize as "cosmetic" issues. A bit ugly, but you can leave with it. The real and unexpected problem appeared as we did first announcement and users came to check the application.

We are using [passport]() for user authorization, which depends on sessions. Each request contains contains a `session_state` which being persisted in cookie. The problem was that, on AppFog for some unknown dropped that `session_state`. So, after user just logged on and clicked somewhere, she was immediately redirected back to login page, as unauthorized.

Absolutely unclear behavior to me.

I definitely know I'm not the [only one](https://www.google.com/search?q=appfog+session+lost&oq=appfog+session+lost) who was suffering that issue. Supported failed to answer quickly and even on my next request they didn't provide anything constructive.

That was the show-stopper with AppFog.

## .. and now Nodejitsu?

Meanwhile, I just deployed application to [Nodejtsu]() and it worked fine there. I was really happy to see the features I missed with AppFog:

* Clear and fast dashboard
* Release management (you can go back and forth with any version you deployed)
* Node.js written CLI and fast deployments
* Good support

The time we joined Nodejitsu it costs 33 USD for 3 instances, that didn't match our needs ideally, but we had to embrace the constraints.

## Conclusions

I know, there is nothing ideal in this world - but if you have choice, it's fine pick up best (or at least better). I would not say AppFog completely sucks, but it didn't work for me as I was expecting.

PS. While writing this post I've noticed that AppFog being [acquired](https://www.appfog.com/savvis/) by CentruLink. I only wish that it would positively affect service and support, so wish good luck for AppFog.
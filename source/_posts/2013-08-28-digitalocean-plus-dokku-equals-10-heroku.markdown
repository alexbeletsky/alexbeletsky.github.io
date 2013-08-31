---
layout: post
title: "Digital Ocean + Dokku = 10$ Heroku"
date: 2013-08-28 22:00
categories: dokku nodejs digitalocean
---

My last infrastructure related [post](http://beletsky.net/2013/07/why-we-moved-from-appfog-to-nodejitsu.html) was about an experience of using AppFog and switching to Nodejitsu eventually. But that was not the end. In short: for [Likeastore](https://likeastore.com/) we needed SSL support, it happed that SSL is only available for Nodejitsu business plan, which price is 120 USD. That was simply to much for our small venture.

Long time ago, I realized - constrains are good. This case just proved that. Looking for alternative options, gave really nice results, which I easily could be re-used if you looking for simple deployment solutions.

<!-- More -->
## Heroku

Heroku is good service. Afraid to be mistaken, but that's probably Heroku who popularized "git-powered" deployments much, ones there deployment script looks like:

```bash
> git push heroku master
```

The rest is all about the service - prepare runtime, deploy code, start web application etc. Besides of that Heroku open-sourced a lot of good stuff, including so called [buildpacks](https://devcenter.heroku.com/articles/buildpacks), some ready to use scripts that are able to setup the dyno with all required runtime to be able to start application there.

I never seriously used Heroku, though. What I dislike, is pricing. Also, the other people I asked about their satisfaction of using Heroku, was not satisfying much (arguable). We needed more lightweight, easy to change setup.

## Digital Ocean

[Digital Ocean](https://www.digitalocean.com/?refcode=de56d081b272) is very fast growing cloud-computing service. It's not PaaS (platform as a service) like Heroku, it's rather IaaS (infrastructure as a service). They are notable for few major things:

* **Pricing** - really (mean, really!) competitive pricing, 1GB RAM, 30GB SSD machine for 10$.
* **Easy of use** - the flow from registration for first droplet creation is smooth and clear.
* **Regions** - machines can be fired up for both US and EU, ideal for us.

But again, Digital Ocean is nothing more as great infrastructure. Herding the server is all up to you. I personally was really afraid of that perspective of setting up nginx, configuring firewalls, load-balancing etc., that prevented me to look to Digital Ocean closely. Getting used to deployment procedures with Nodejitsu and Heroku, it's real pain to deploy app by FTP again and do everything manually.

But lucky chance I noticed [Dokku](https://github.com/progrium/dokku) project and that was something really great, explain why later.

## Dokku & Docker

So, [Dokku](https://github.com/progrium/dokku) is simply amazing hack (or more correctly, combination of diffrent hacks) by [Jeff Lindsay](https://github.com/progrium). Some initial facts:

* Written in Shell script and currently nearly 100 lines of code
* Based on Docker
* Provides Heroku-like deployment experience
* Community-driven

Dokku, can be installed to Ubuntu 13 machine and turn that machine into Heroku-like server. I mean, you can push the code there and Dokku, will fire-up new *docker process*, deploy code there, start the application, configure nginx and setup environment variables and SSL.

Btw, the time I just looked to Dokku, they missed exactly support of ENV variables and SSL. It was not acceptable for me, without those 2 features. Constraints again, but that gave me ability to contribute the projects and eventually I submitted both features.

Dokku, is very interesting project. First of all, because it based on trendy [Docker](http://www.docker.io/). Docker is "Virtual-Machine-Based-Deployment" alternative. Deployment on virtual boxes are de-facto standard now and docker is about to change that. Guys from [dotCloud](https://www.dotcloud.com/) open-sourced solution that allows to run isolated processes (containers) - that are like lightweight virtual machines. You can deploy docker on Ubuntu server and then use that as host any kind of applications or databases.

Docker could turn Ubuntu Server to PaaS and Dokku makes great "interface" for that.

Each Dokku piece is very interesting indeed and I hope to blog more about. And Dokku is using Heroku buildbacks, which makes you feel you deal with Heroku, not with your own setup.

## Putting Things Together

Digital Ocean and Dokku make a perfect match. As I said above, Digital Ocean is something you can really start quick with. So, what we did is just started up 10$ Ubuntu 13 server and installed Dokku there. In total it took 7 minutes or so. I would not bother you with instructions, cause you find a lot in [internet](https://medium.com/code-adventures/438bce155dcb).. but also, assuming that DO + Dokku is a kind of Apple product, that does not require instructions.

First impression was simply amazing. You have everything under control and fill great with "git-powered" deployments. So, after successful try that machine became our staging server and I also fired-up another one for production one.

Now, then we developing features and what to show each other or test, you just need to do following:

```bash
> git push deploy-staging feature:master
Counting objects: 7, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 445 bytes | 0 bytes/s, done.
Total 4 (delta 2), reused 0 (delta 0)
remote: -----> Building collector ...
remote:        Node.js app detected
remote: -----> Resolving engine versions
remote:        Using Node.js version: 0.10.15
remote:        Using npm version: 1.2.30
remote: -----> Fetching Node.js binaries
remote: -----> Vendoring node into slug
remote: -----> Installing dependencies with npm
remote:        npm WARN package.json likeastore-collector@0.0.2-21 No repository field.
remote:        npm http GET https://registry.npmjs.org/mongojs
remote:        npm http GET https://registry.npmjs.org/underscore

...


remote: =====> Application deployed:
remote:        http://stage-collector.likeastore.com
```

The time we are ready to release:

```bash
> git push deploy-production master
Counting objects: 7, done.
Delta compression using up to 8 threads.
...


remote: =====> Application deployed:
remote:        http://collector.likeastore.com
```



It's fast and it's pretty reliable.

For conclusion, I would say that using both Digial Ocean and Dokku was a clear win for [Likeastore](https://likeastore.com/) being released.
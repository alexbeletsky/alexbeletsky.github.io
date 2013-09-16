---
layout: post
title: "PaaS in Your Pocket with Dokku"
date: 2013-09-16 21:21
comments: true
categories: docker dokku nodejs conference transcript
---

This is transcript of talk I gave on [RejectJS]() conf in Berlin, September 2013.

What I'm going to talk about is Dokku - pocket-size, very cool and interesting project, that would make you (hopefully) re-think of the code shipping.

<!-- More -->

<script async class="speakerdeck-embed" data-id="d912a0f0fdbc0130260d1ebd49b9b82c" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

## Background

So, some background on me. My name is Alexander and I live in Kiev and work for Danish company, named E-conomic. There we do, we interesting product called Debitoor and a lot of cool stuff happens there.. But besides of that full-time job, with a friend of mine, I'm doing a small project called Likeastore.

And this is exactly, side-project are something **there you learn some new things and you try these things**. So, it's started out as hackathon project, it's Node.js, MongoDB, Angular.js - so highly JS related thing (even if my talk is not so JS oriented). On a hackathon we got really exited on the way it went, soon we realized that we want to push things out. So, we wanted to release, and conquer the world with the cool idea we had.

Probably the most important lesson I've learned in whole my career - the way you ship the code, matters!

It matters a lot, but the key fact - as faster you can ship the code from machine A to machine B, as faster you running business as happy team you have. So, if you shipping once a 2-weeks, you are fine, if you shipping 1-week are are great, if you shipping 1-per-hour you are superb (if you don't believe me, take a look on github guys, how happy they are all the time).

And long time I realized that deployment of code, should be as easy as:

```
git push origin master
git push heroku master
af deploy
jitsu deploy
```

Whatever, important is - one click (command, shell script) you are done.

So, we were seeking for good deployment and hosting options. And I love Node.js in particular for it's "deployment-friendly" environment.

Code deployment in general is very known developers problem. It solved many time and with cloud computing it **evolutionized in something that's know as PaaS (platform as service)**. And PaaS is something that brings a lot of value for any project, since you abstract out the exact infrastructure, and what you do is write code push it out, the rest is solved by PaaS.

Of cause, we started to look for PaaS that fits us best.

Started with AppFog, jumped to Nodejitsu, but met few constrains that forced us to gave up. Namely, that we had to have SSL connection, since we dealing with users private data, but Nodejitsu is proposing SSL for 120 USD that, simply too much of the project you begin with and don't have great budget behind.

I was really unhappy that time, since I realized a lot of overhead work that comes to you, then you drop PaaS.

This is where constraints coming from, and sometimes they are very unpleasant, but in general constraints are good. With perfect timing, I've seen Dokku project.

Even if I said Dokku is small project, it's standing on shoulders of giants, and those giants are Docker & Builtpacks & Gitrecieve (there is a few more, but those are most important ones).

During this talk, I'll highlight both Docker and Dokku internals, but my primary goal is to rather share the **experience of using Dokku that helped to making the things done**.

## Dokku

Dokku is created by bright hacker - Jeff Lindsay. And what Jeff did is he saw the opportunity of mixing all the pieces together in one tasty cocktail.

Dokku relying on Docker for running containers (as isolated application instance), it uses Gitrecive to implement git interface for server, and it's relying on Buildpacks, for preparing environment for container.

Now, it's time to show you how it works in reality:

```
demo with node-sample
```

## Gitrecieve

Gitrecieve is a component which is able to recieve you git push command, create repository on fly and trigger recieve script. Recieve script could be anything you want.

## Buildpacks

Buildpacks are open source components released by Heroku. How many of you guys ever used Heroku? All right, so the output you see, while pushing code to Heroku is exactly the **result of work by Buildpack**.

Buildpack is responsible for few things: it detects the environment of you app (like if I can see `package.json`, that would probably mean Node.js app), downloads all required runtime components and run build script. Once it run, we have created infrastructure where application is ready to start up.

So, Heroku created a lot of buildpacks, for Ruby, Python, Java, etc., and all that goodness is reused by Dokku.

## Docker

How many of guys seen the videos, or read the materials about Docker? I hope many of you, because it's kind on noisy now.

So, Docker in essence, is the engine of **portable containers**. Portable? Containers? You are probably very interesting what actually goes, behind these words.

Docker is open-source project by dotCloud. And the dotCloud is actually specializing on shipping the code. Company's co-founder Salmon Hykes, gives are very nice metaphor on containers: imaging a shipping of goods like, 100 years ago - where were ships and barrels and bags and boxes, etc. Because of that variety you had a different options for shipping. In middle 1950, the idea of container born. Container defines size and dimensions, but in essence - you can put what ever you like there, close the door and that's it. It's guaranteed that you container is shipped, because all equipment fits it.

So, in dotCloud they've developed a fairly complex infrastructure, that would allow to run their business. After 5 years of hacking that, they reached **great level of expertise**, as well they realized the need to take out the core system, rewrite it completely and open source it. That was then Docker project born and if Salmon would ever listen to my talk, I would like to give tons of Kudos for him, for that job.

Docker itself is hack above the Linux kernel technology called LXC. And LXC is the of running isolated processes. LXC is a Linux process, that have it's own file system, networking interfaces, processes etc.

Sure, you might think - it smells like virtual machines, why containers are better?

Image container is immutable entity. Once the command run, it's changed.. but as soon as change is not committed the image remain the same. That mean, you can re-use some existing image as many time as you want, all the time starting up, so called clean environment. The clean environment is something we like of using VM.

Startup time of container is **many times faster, then booting up new VM image**. And this is it's primary benefit.

Besides of that, Docker comes with so called Index. Index is a public repository of ready to use images. Different operations systems and different pre-installed software is there.

## Nginx

All HTTP / HTTPS orchestration is done by Nginx.

## Deploying the applications with Dokku

So, we've came to the most interesting part, how Dokku is **helpful for the product development**. Theoretically, Dokku can work on any Linux machine with LXC support, to run docker, dokku itself is written on shell script, which make it really portable. Practically it works on Ubuntu 13 64b.

You build you own small PaaS on any IaaS as you want. One's which is able to fire up Ubuntu machine and give you ssh access to it. For a few reasons, I've picked up Digital Ocean, since they indeed provide really nice service and competitive prices.

Once the machine is started, you push have to install Dokku there. After it's installed, you upload you SSH key there, so you are able to do git push.

You are able to configure environment variables, as well as configure SSL connection.

That's a **perfect fit**, especially for small projects.
---
layout: post
title: "Playing with Dokku on Vagrant"
date: 2013-09-03 19:57
comments: true
categories: dokku docker nodejs vagrant
---

As I said [previously](http://beletsky.net/2013/08/digitalocean-plus-dokku-equals-10-heroku.html), it's very easy to turn Linux machine into Heroku-like server. But, before setting up paying account on Amazon or Digital Ocean, it's nice to just play it locally. Will do that, running a Dokku on virtual machine. Will setup development environment and do first local deployment, just to see some real features.

It does not require complex environment to run Dokku locally. All you need is `git` and `Vagrant`.

<!-- More -->

## Prepearing Environment

You box should contain few things: git (github account), VirtualBox and Vagrant. If you don't have Vagrant installed, please do now. It makes a lot of sense to keep such kind software on machine.

Here you can find [some instructionn](http://docs.vagrantup.com/v2/getting-started/index.html) of how to do that.

### Cloning Dokku

You should clone Dokku repo locally. In your development folder, run

```bash
> git clone git@github.com:progrium/dokku.git
> cd dokku
```

Dokku repository already contains Vagrant file, with all required configuration.

### Local Networking

Just in sake of convenience, will map `10.0.0.2` ip address (the one that Vagrant machine is assigned with), to `dokku.me` DNS name, so it's just handy for testing.

```bash
› sudo nano /private/etc/hosts
```

and put last 2 lines, as it shown in my example:

```plain
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
fe80::1%lo0     localhost
10.0.0.2        dokku.me
10.0.0.2        node-simple.dokku.me
```

### Fire Up Virtual Machine

In `dokku` folder, you should run

```bash
> vagrant up
```

It will start to prepare new virtual environment for you,

```bash
 vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] -- 80 => 8080 (adapter 1)
[default] Running any VM customizations...
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
```

I have a problem with this step few times, so if you machine could not be booted, try to run `vagrant reload`, it should help.

It would took about up to 20 minutes, to fire up virtual machine, install git there, install Docker + Dokku. As soon as it's done, it's possible to access machine by `ssh`.

Last thing you need to do, is to upload your `ssh` key to Dokku server, so you will be to `git push` code there.

```bash
> cat ~/.ssh/id_rsa.pub | ssh root@dokku.me "sudo gitreceive upload-key alexanderbeletsky"
```

You can use default vagrant root password, which is `vagrant`.

Now, just to check that everything is fine simply access you machine by `ssh`,

```bash
> vagrant ssh
```

So, just try that everything is running fine by checking version of Docker:

```bash
> docker -v
Docker version 0.6.1, build 5105263
```

The instance is ready for deployment.

## Deploy to Dokku

If you still there, you can just quit vagrant `ssh` session, and went to the folder with Node.js application. I'll be using really simple [app](https://github.com/alexanderbeletsky/node-simple), called `Node-simple` - express.js based, that serves one HTML file and shows `NODE_ENV` env. variable.

So, what you need to setup is remote repository to push to:

```bash
> git remote add local-deploy git@dokku.me:node-simple
```

That's all. You ready for first deploy, just push code to machine with Dokku:

```bash
git push local-deploy master
› git push local-deploy master --force
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 289 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
-----> Building node-simple ...
       Node.js app detected
-----> Resolving engine versions
       Using Node.js version: 0.8.25
       Using npm version: 1.2.30
-----> Fetching Node.js binaries
-----> Vendoring node into slug
-----> Installing dependencies with npm
       npm WARN package.json application-name@0.0.5 No repository field.
       npm WARN package.json application-name@0.0.5 No readme data.
       npm WARN package.json send@0.1.0 No repository field.
       ...
=====> Application deployed:
       http://node-simple.dokku.me

To git@dokku.me:node-simple
   dd05aae..ac5b6da  master -> master
```

## Setting up Environment

Every application requires environment. It's common practice to set at least `NODE_ENV` variable for Node.js applications. To do that, you need to create `ENV` file inside the `/home/git/node-simple` folder.

```bash
› ssh root@dokku.me "echo export NODE_ENV="development" > /home/git/node-simple/ENV"
```

Now, let's re-deploy the application, change the version in `package.json` and push code again.

```bash
› git push local-deploy master
```

Now, application is ready to be accessed.

## Accessing the Application

Open Chrome and hit `http://node-simple.dokku.me`, so you will see such response:

{% img /images/blog/node-simple-run.png %}

You can play a bit further, by just changing some Node.js code of end-points and re-deploy. Each time, new docker instance is started and served on `http://node-simple.dokku.me`. The experience of deployment is really like something you have with Heroku.

Just looking on logs while new application is deployed, would give pretty much idea of what's going on there.

So, your local Vagrant image will good start place, before you ready to use Dokku on cloud.


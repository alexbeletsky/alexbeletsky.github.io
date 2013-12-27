---
layout: post
title: "Run Several Processes in Docker Container"
date: 2013-12-27 14:15
comments: true
categories: docker seismo mongodb nodejs
---

What I like the most about [Docker]() project is new opportunity to deploy and distribute software. Many times I've been to situation when I wanted to play with some software and get exited about, but after I read installation manual my excitement totally gone. Non trivial applications, requires quite a lot dependencies: runtimes, libraries, databases.

With docker, the installation instruction got reduced to something like:

```
$ docker pull vendor/package
$ docker run vendor/package
```

Simply like that, forget about missing Java Runtime on your server. It suits perfectly for TCP/HTTP applications.

<!-- More -->

Being messing around [Seismo](https://github.com/seismolabs/seismo) project I realized, I want to go exactly same way. Since it has few dependencies now, MongoDB and NodeJS - it should be easier to anyone to try it, even if they do not use that setup. I was happy to see, that GitHub currently offers great support for Docker. Namely, if you have repo with `Dockerfile` inside, each time you push the code, docker image got rebuild and pushed to public [index](https://index.docker.io/).

I've created `Dockerfile` that would build up image, ready to have Seismo run inside.

```plain
FROM    ubuntu:latest

# Git
RUN apt-get install -y git

# MongoDB
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list
RUN dpkg-divert --local --rename --add /sbin/initctl
RUN ln -s /bin/true /sbin/initctl
RUN apt-get update
RUN apt-get install mongodb-10gen
RUN mkdir -p /data/db

# NodeJS
RUN apt-get update --fix-missing && apt-get upgrade -y
RUN apt-get install -y wget curl build-essential patch git-core openssl libssl-dev unzip ca-certificates
RUN curl http://nodejs.org/dist/v0.10.22/node-v0.10.22-linux-x64.tar.gz | tar xzvf - --strip-components=1 -C "/usr"
RUN apt-get clean && rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

# Seismo
RUN git clone https://github.com/seismolabs/seismo.git /seismo
RUN cd /seismo; npm install
ENV PORT 8080
EXPOSE 8080

WORKDIR /seismo
ENTRYPOINT ["./bin/run.sh"]
```

It's based on latest Ubuntu server, installs Git, MongoDB and NodeJS runtime and clones Seismo itself inside image.

But, I've met a problem to start few processes inside the container. Since I need both MongoDB for storage and NodeJS for API server, it's required both be running inside one container. If shell script just starts one, `mongod` for example, `node app.js` is not executed.

I was a little worried, thinking it's not possible to run more that one process inside container.

But solution was found. I've created another shell script that starts `mongod` as background process and starts `node` after.

```bash
#!/bin/bash
mongod & node ./source/server.js
```

That worked as charm.
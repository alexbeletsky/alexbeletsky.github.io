---
layout: post
title: "We Were Wrong 10 Years Ago"
date: 2013-06-20 23:43
comments: true
categories: mind technologies
---

The way we build distributed systems and platforms is changing through the the last 10 years. Recently, I was thinking myself about different technological options I used so far and came for some conclusions.

Early 2000's I was programming C/C++ and the best way of passing some data from one machine to another, was - *socket*. The time took understanding of sockets, I clearly saw it's the best solution for that task.

<!-- more -->

Sending and receiving strings and binary structures was very nice comparing to using C or assembler to directly talk to network driver. Sockets allowed to build server-client architectures, in theory allowing application written in different languages to communicate each other.

But time passed and COM / DCOM technologies appeared. Data transport operations was completely hidden, a lot of abstraction put above objects and marshaling. Again, that technology was massively adopted by developers, many projects were created based on COM / DCOM (or similar technology called CORBA).

Marshaling, IDL, ATL and other stuff was very difficult to understand and to use. But still there was much better than implement something similar on raw sockets.

Era of Web Services has begun. Web Services indeed allowed build highly distributed systems based on reliable HTTP protocol. Data transport mechanisms appeared to be transparent and very clear. Data transfered back and forth as XML payloads. Almost all web service technologies were build in concept on [RPC](http://en.wikipedia.org/wiki/Remote_procedure_call).

[SOAP](http://en.wikipedia.org/wiki/SOAP) was primary payload format, formalized XML with metadata inside. Again, previous DCOM / COBRA things looked very ugly, Web services looked very promising.

Web Services, definitely affected the way we architecture distributed systems now. It formalized a lot, `WS-*` set of standards came out. Many vendors like Microsoft or Sun created bunch of frameworks to support it. Many books, many blog posts and speaks issued on that topic.

Nowadays, `RPC-based` services completely substituted with `REST-style`. XML as payload, seems to be awful, cause its verbose, requires more bandwidth comparing to `JSON`. `JSON` in conjunction with dynamic languages gives great framework of building distributed systems. We are currently in era of *API oriented* systems and as soon as you adopted that, SOAP and RPC makes no sense to you.

What I'm telling is, technologies are improving very fast. What was great today, will appear awful tomorrow.

We were wrong 10 years ago that some particular technology is cool. So now, than I see some new and shinny thing I'm no longer blindly excited about that. It will gone... and substituted with something better.

Developers ultimate goal remains to *influence* that process. And make influence is easy - share your thoughts, implement prototypes, talk to people.

Make things simpler, make things better.
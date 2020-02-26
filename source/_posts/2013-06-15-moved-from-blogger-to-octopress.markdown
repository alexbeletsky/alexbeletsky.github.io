---
layout: post
title: 'Moved from Blogger to Octopress'
date: 2013-06-15 16:18
comments: true
categories: life,mind,open source,github
---

I actively started to blog almost 3 years ago. Before that, far away 2008 I created blogger account and had few posts there about TDD and ASP.NET development. So, after some pause I just picked up existing one and jumped in in blogging community.

That time, blogger was quite obvious choice. Google powered, editor and plain HTML support, templates etc. I used to use Visual Studio to create posts, then pasted that to blogger, posting images on Picassa and code on Github. That worked for quite while, but now I consider that is very heavyweight approach.

Now this site is running [Octopress](http://octopress.org/) and hosted on [Github](https://github.com/) and that's totally reflects my current tools of choice.

<!-- more -->

## Reasons to move

My primary concern was up to blog design. The time I started to blog I had almost zero skill to HTML/CSS, so picking up some existing theme was best option. In 2010 [beletsky.net](http://beletsky.net) looked pretty fine for me. But time has passed, my vision of design has changed, requirements for blog UI has changed (consider mobile devices), the way I blog has changed.

I tried to tweak blogger theme and tried to create new one, but that appeared a _bloody_ mess.

Besides the design, the recent Google actions like closing up Picassa, Reader - are quite obvious signs that Blogger days are counted. They would probably try to integrate it to Google+ services and I'm literally sick of Google+.

Finally, with much of Github influence, I [realized](http://alexbeletsky.github.io/2013/05/github-as-blogging-platform.html) Markdown is awesome, blogging in plain HTML sucks hard. I wanted to have nice and easy platform for handling my blog and personal page.. I wanted to blog like [hackers](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html) do.

## How to move, easy way

I was really _afraid_ of difficulties related to moving to other platform, that's why I procrastinated that so long.

That lead to the point then I wanted to blog about something, but running up VS, starting HTML coding and uploading images somewhere simply prevented me to do that. Blogging should be as easy as writing to notepad, in that way you will do that with pleasure.

But, I was really surprised how **simple** it actually was. I found that small and precise blog [post](http://blog.thepete.net/blog/2012/02/08/blogger-to-octopress/) by [Pete Hodgson](http://twitter.com/ph1). After reading some docs on [Octopress](http://octopress.org/) site I had running setup in 10 minutes or so. That was great start.

But I didn't want to have default theme of Octopress, I didn't want to pick up [3rd parties](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes) as well. My goal was to create something original.

## Custom theme for Octopress

I've picked up just released [Pure CSS](http://purecss.io/) framework, since I _promised_ myself never ever create my custom grids and layouts, since it works really bad. I basically use only grid system from there.

I wanted to have something clean, minimal, black & white theme. My inspiration was primarily from [zachholman.com](http://zachholman.com/), [blog.brunoscopelliti.com](http://blog.brunoscopelliti.com/), [carmo.org.uk](http://carmo.org.uk/).

So, I spent maybe 2-3 hours to sketch new blog look.

{% img /images/blog/design-blog-sketch.png [beletsky.net design sketch [design sketch]]%}

I showed that to colleagues and received some valuable feedback from [@voronianski](http://twitter.com/voronianski) regarding fonts and styles. Sketch looked amazingly good on Mac retina, with custom fonts (Open Sans, Lato). It has anti-aliasing issues on Windows, though.

Turning resulted HTML / CSS into Octopress was **not** so easy.

I've spent up to 16 hours to make it work nicely. There was a lot of CSS style conflicts between existing SASS and my custom one. So, I had to disable almost everything and use my custom `style.css`. But still, that was worth effort if you compare with previous look,

{% img /images/blog/old-beletsky-net.png [beletsky.net old look [old look]]%}

Since I had some heavily read posts, I need to make sure that old google indexed URL's would work. Fourtunatelly, that was really easy to configure. In `_config.yml`, just paste:

```
permalink: /:year/:month/:title.html
```

That would give **exactly** the same URL as blogger have. Even if I hate that `.html` extension, nothing I can do now.

I've turned [beletsky.net](http://beletsky.net) a bit more toward personal page, not only blog. There are about me, talks and projects page, that would demonstrate the profile more clearly I hope.

And I dropped comments. I don't receive many and if you want to get in touch, twitter and github are better alternatives.

## Hosting and domain names

Another problem was to get my custom domain back from Google. I've bought it tougher with Google Apps, which I never actively use, except creating some email boxes to workaround some XBox Live stupidity.

I was mad about I'm not able to control DNS configuration of my own domain name. `Whois` showed that `beletsky.net` is actually registered by [enom](http://www.enom.com/) registrar. So, I dropped a line there and with my great surprise that replied quite fast. We agreed, I move the domain name to another registar that I use for other stuff I own.

That took a lot of time (more than 10 days) and even now DNS configuration is not yet broadcasted.

Nevertheless, hosting Octopress blog on Github is best option. And Octopress has build in `rake` tasks for that.

## Happy blogging time

So, the experience is different now. No more HTML, only clean and nice Markdown.

I do everything in [Sublime Text 2]() (wring post in _distraction free_ mode is great) that has good syntax highlighting for Markdown and spell checker, from the box.

Octopress running preview mode, so with each save of file you can check out the results on locally run web server. No more uploads of images to Google+, simple placing on file system.

Nice integration with `gist` so I hope to do more code shares there. Code coloring is very nice, but I hope to customize that in future as well.

New UI makes my eyes happy and this is big motivator to produce more content.

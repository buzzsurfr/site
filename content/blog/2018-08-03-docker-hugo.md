---
title: "Docker Hugo"
date: 2018-08-03T09:00:00-05:00
draft: false
categories: ["Deep Dive"]
tags: ["docker", "github", "hugo"]
description: "After [restarting my blog](/2021/11/01/blog-restart/), I wanted a way to automate my workflow. I currently work for AWS, and want to use the features of the ..."
image: ""
---

After [restarting my blog](/2021/11/01/blog-restart/), I wanted a way to automate my workflow. I currently work for AWS, and want to use the features of the cloud to manage and deploy my blog, but for as little cost as possible. The lowest cost for a static site like mine is Amazon S3, which offers to host the objects in the bucket as a static website.

This starts by adopting a solid framework for building static sites. After trying a few, I selected [Hugo](http://gohugo.io/). I had been using [mkdocs](https://www.mkdocs.org/) for training/tutorials but felt it lacked a good native layout engine and wasn't a good fit for a blog.

I followed the [installation instructions](http://gohugo.io/getting-started/installing/), but wanted something I could *containerize* (since it's relevant to my current work). Thus, I created [docker-hugo](https://github.com/buzzsurfr/docker-hugo) as a simple project to containerize hugo.

For now, this includes a [README](https://github.com/buzzsurfr/docker-hugo/blob/master/README.md) and a [Dockerfile](https://github.com/buzzsurfr/docker-hugo/blob/master/Dockerfile) *(copied as of August 3, 2018)*:

```
FROM centos:latest as builder

RUN yum -y update
RUN curl -sL -o hugo.tar.gz https://github.com/gohugoio/hugo/releases/download/v0.46/hugo_0.46_Linux-64bit.tar.gz && tar zxf hugo.tar.gz hugo

FROM scratch

COPY --from=builder /hugo .

VOLUME /host
WORKDIR /host
EXPOSE 1313

ENTRYPOINT ["/hugo"]
```

While a simple example, it does combine some newer Docker features. I used a multi-stage build to download the actual binary, then a *scratch* image for the actual deployment. The [README](https://github.com/buzzsurfr/docker-hugo/blob/master/README.md) highlights the syntax I use for the command, and an alias for being able to run `hugo new posts/docker-hugo.md` with all of my environment variables already plugged in. This can also be adapted for a future CI/CD process.
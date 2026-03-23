---
title: "Securing Container Workloads on AWS Fargate"
date: 2019-02-19T19:56:00-05:00
draft: false
categories: ["Deep Dive"]
tags: ["aws", "fargate"]
description: "When containers first became mainstream (think [PyCon 2013](https://us.pycon.org/2013/) with [Solomon Hykes on stage](https://www.youtube.com/watch?v=wW9CAH9..."
image: ""
---

When containers first became mainstream (think [PyCon 2013](https://us.pycon.org/2013/) with [Solomon Hykes on stage](https://www.youtube.com/watch?v=wW9CAH9nSLs)), everyone thought it had potential and began to test running containers on their own, but almost no one set out to put containers in production that day. They wanted to see it *battle-tested*...which has happened over time. Containers have matured from an *emerging technology* to *production-ready* where it's generally considered safe, but there's a new problem. Now, we need our business processes, tools, and architecture models to mature as well.

The top ask I hear about containers comes down to security. Containers were built as a way to isolate workloads for one another, but many of the security models from virtual machines do not work in containers, and thus we must evolve our thought process.

To that end, I presented an [AWS Online Tech Talk](https://aws.amazon.com/about-aws/events/monthlywebinarseries/) about how to secure container workloads using AWS Fargate (though many of the lessons also apply generally across containers). I demonstrated some quick steps to make your containers more secure during the build process as well as how to enhance visibility and security around containers running in AWS Fargate.

https://www.youtube.com/watch?v=PkR2hu1HCCY
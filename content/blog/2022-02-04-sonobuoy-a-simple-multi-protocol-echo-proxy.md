---
title: "Sonobuoy - a Simple, multi-protocol echo proxy"
date: 2022-02-04T11:45:21-05:00
draft: false
categories: ["Build Log"]
tags: ["IT"]
description: "A few customers that use [AWS App Mesh](https://aws.amazon.com/app-mesh/) want a way to ensure that the Virtual Gateway is *properly* routing, not just up an..."
image: ""
---

A few customers that use [AWS App Mesh](https://aws.amazon.com/app-mesh/) want a way to ensure that the Virtual Gateway is *properly* routing, not just up and available. The envoy for the Virtual Gateway provides a health check, but requires in-depth knowledge and observability to determine whether the proxy is successfully routing traffic. Instead, create a simple route to `/health` or `/ping` and send it to a *known, working* service.

There's a plethora of different options to use for the backend. Some setup a nginx/envoy proxy to respond to packets where others will use a clone of a microservice. Instead, I wrote my own.

Introducing [sonobuoy](https://github.com/buzzsurfr/sonobuoy). Written in Go, sonobuoy can be deployed as an agent or a sidecar container and supports TCP, HTTP, and gRPC protocols to provide the cleanest successful response to a request. 

Here's an example, with the TCP listener on port 2869, the HTTP listener on port 2870, and the gRPC listener on port 2871:

[![](https://asciinema.org/a/466717.svg)](https://asciinema.org/a/466717)
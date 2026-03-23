---
title: "Cheap and quick Mastodon alias"
date: 2022-11-10T21:04:51-05:00
draft: false
categories: ["Quick Tip"]
tags: ["IT"]
description: "**EDIT:** The format is JRD+JSON per [RFC 7033](https://www.rfc-editor.org/rfc/rfc7033#section-10.2). Changed the reference below, and thanks to [mdaniel](ht..."
image: ""
---

**EDIT:** The format is JRD+JSON per [RFC 7033](https://www.rfc-editor.org/rfc/rfc7033#section-10.2). Changed the reference below, and thanks to [mdaniel](https://news.ycombinator.com/user?id=mdaniel) on HN.

With the uncertainty of Twitter looming over us, I did what everyone else in the community did and looked to alternatives, including [Mastodon](https://joinmastodon.org). The appeal of Mastodon is the distributed nature, but that's also a pitfall to *muggles* (non-technicals).

I want to use a simple alias for being able to find my Mastodon name, and went and purchased `salvo.chat`. I have a number of Twitter aliases (because of course I do!) so I wanted something incredibly simple. Unfortunately, most of the Mastodon hosting providers are completely overwhelmed right now as we all figure out how to do with the influx of demand.

I did consider setting up a Mastodon server, but I definitely started to *over-engineer* it (was gonna host it in Kubernetes on my homelab) so instead I changed gears and said "what can I do *fast* that's a temporary alias?"

There's been a few technologies that I've been looking for a use case. I need something now that's cost-predictable, simple, and easy to setup--and put these together real quick:

- DigitalOcean droplet (1 vCPU, 512 MB RAM, 10 GB SSD, $4/mo)

- Caddy server

The droplet was simple enough, and I get a credit for two months (somehow I've never used DO before) which gives me time to customize and find a long-term solution. However, the exciting part is using [Caddy](https://caddyserver.com). It's written in Go and includes some nice features including automatic HTTPS. This means with almost NO configuration I can have a secure website that will alias to my mastodon alias.

I wasn't really sure *what* to do to get it working though, but fortunately I came across [Mastodon on your own domain without hosting a server](https://blog.maartenballiauw.be/post/2022/11/05/mastodon-own-donain-without-hosting-server.html) by Maarten Balliauw which walked me through the technical details. I took his discovery and used it to setup my alias server.

## Steps to recreate

First I had to get my `webfinger` details from the current provider--a simple cURL helps here.

```
`$ curl https://mastodon.cloud/.well-known/webfinger?resource=acct:buzzsurfr@mastodon.cloud
{
    "subject": "acct:buzzsurfr@mastodon.cloud",
    "aliases": [
        "https://mastodon.cloud/@buzzsurfr",
        "https://mastodon.cloud/users/buzzsurfr"
    ],
    "links": [
        {
            "rel": "http://webfinger.net/rel/profile-page",
            "type": "text/html",
            "href": "https://mastodon.cloud/@buzzsurfr"
        },
        {
            "rel": "self",
            "type": "application/activity+json",
            "href": "https://mastodon.cloud/users/buzzsurfr"
        },
        {
            "rel": "http://ostatus.org/schema/1.0/subscribe",
            "template": "https://mastodon.cloud/authorize_interaction?uri={uri}"
        }
    ]
}`
```

I also pre-built a droplet and set my DNS for the domain to point to the droplet.

I then saved this to a file in the droplet, and moved to [installing Caddy](https://caddyserver.com/docs/install). I went the package route so I could make quick updates if necessary, then had to find the `Caddyfile` (which was in `/etc/caddy`). The Caddyfile has enough to launch a web server locally. The only changes I had to make was to change the listener to the domain (which enables automatic HTTPS) and added the header so that the `webfinger` response would be JRD+JSON. I'm not sure it was necessary, but when you work on load balancers as I have, you want to make sure.

```
`# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

salvo.chat {
	# Set this path to your site's directory.
	root * /usr/share/caddy

	# Enable the static file server.
	file_server

	# Another common task is to set up a reverse proxy:
	# reverse_proxy localhost:8080

	# Or serve a PHP site through php-fpm:
	# php_fastcgi localhost:9000

	route {
		header /.well-known/* Content-type application/jrd+json
	}
}

# Refer to the Caddy docs for more information:
# https://caddyserver.com/docs/caddyfile`
```

A quick restart, and my server was running. I tried cURL on the new URL:

```
`$ curl https://salvo.chat/.well-known/webfinger?resource=acct:buzzsurfr@mastodon.cloud
{
    "subject": "acct:buzzsurfr@mastodon.cloud",
    "aliases": [
        "https://mastodon.cloud/@buzzsurfr",
        "https://mastodon.cloud/users/buzzsurfr"
    ],
    "links": [
        {
            "rel": "http://webfinger.net/rel/profile-page",
            "type": "text/html",
            "href": "https://mastodon.cloud/@buzzsurfr"
        },
        {
            "rel": "self",
            "type": "application/activity+json",
            "href": "https://mastodon.cloud/users/buzzsurfr"
        },
        {
            "rel": "http://ostatus.org/schema/1.0/subscribe",
            "template": "https://mastodon.cloud/authorize_interaction?uri={uri}"
        }
    ]
}`
```

And that's it! Now if you go to your Mastodon client and search for `@theo@salvo.chat` my `@buzzsurfr@mastodon.cloud` comes up!

[![](https://theodorejsalvo.files.wordpress.com/2022/11/mastodon_cloud_alias.png?w=1024)](https://theodorejsalvo.files.wordpress.com/2022/11/mastodon_cloud_alias.png)
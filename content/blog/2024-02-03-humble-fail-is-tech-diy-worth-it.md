---
title: "Humble Fail: Is tech DIY worth it?"
date: 2024-02-03T13:11:53-05:00
draft: false
categories: ["Honest Review"]
tags: ["astro", "github", "rpg", "splash"]
description: "I broke my splash site, and learned a few lessons about community, build vs buy, and sharpening the saw......"
image: ""
---

## What happened?

I run a [splash site](https://cyberdru.id) to easily link to all my profile sites (think LinkTree but I own it). The site is built on [Astro](https://astro.build) and uses [tailwindcss](https://tailwindcss.com) and [Astro Icon](https://www.astroicon.dev). 

I was in the process of adding new profile links for D&D Beyond and Roll20. I could run the development server locally and everything worked, but the public site wasn't updating. I go to check my GitHub Actions logs and find this message:

```
`Node.js 16 actions are deprecated. Please update the following actions to use Node.js 20: actions/checkout@v3, actions/setup-node@v3. For more information see: https://github.blog/changelog/2023-09-22-github-actions-transitioning-from-node-16-to-node-20/.`
```

I follow the link and read on, the change is as simple as changing my workflow to use v4 and specifying Node.js 20. I push the change, only to find that it broke again. This time it was the astro-icon module. I was running v0.8.1 so I upgraded to v1.1.0 (following [instructions](https://www.astroicon.dev/guides/upgrade/v1/)), but it kept failing. I spent hours searching for a fix, and finally was going to file a bug report on astro-icon, when I came across their issue template that reads:

> 

✅ I am using the **latest version of Astro Icon**.
✅ Astro Icon has been added to my `astro.config.mjs` file as an `integration`.
✅ I have installed the corresponding `@iconify-json/*` packages.
✅ I am using the **latest version of Astro** and all plugins.
✅ I am using a version of Node that Astro supports (`>=18.14.1`)

Source: https://github.com/natemoo-re/astro-icon/blob/main/.github/ISSUE_TEMPLATE/bug.yml

I'm typically quick to dismiss these, but my days in support meant that I had to run through each of them. I get down to "the latest version of Astro". I'm running v2.9.7--which sounds like the latest version. Out of curiosity, what's the latest version?

v4.3.2 🤬

Sure enough, upgrading fixed it.

## What did you learn?

There were a few takeaways from my Saturday morning shenanigans:

1. **"Build over buy" still has consequences.** I didn't want to pay Linktree's subscription, thinking I could maintain the site cheaper. I still think that's a good choice, but had I used LinkTree then I wouldn't have lost this Saturday morning. I also chose to build my own because of the level of customizations I wanted/plan to add. I chose "build over buy", and got burned 🔥 (a little).

2. **"Sharpening the saw" only works if you keep up with it.** I chose Astro after seeing someone else's site built that way (sorry don't remember who) and I liked the simple use of tags to convey intent, then putting the design of it (which is repeated many times) into a separate file. I don't use Astro for anything else, and that directly contributed to how long I spent on the issue. I feel that I would have solved this almost immediately had I spent more than 20 minutes every 6 months using Astro.

3. **Corporations go through this on a much larger scale.** My problem is IDENTICAL to major corporations who invest in agile development, then ignore the practices. If I spent more time working on this splash site, I would've kept it updated and would have built up the experience to know to check for the latest version. When I didn't--I spent a lot of time trying to figure out why.

4. **Good community hygiene works.** I now avoided filing an issue on a project because of Astro Icon's issue template. I don't see good issue templates often, but this one was concise and direct...and showed me the problem.

Ultimately, this problem got me thinking about whether DIY in tech is worth it. I don't think I considered troubleshooting time when I decided to "build", but I still like the end result and will continue to build my splash site. I've also gone back on this blog between Hugo and Wordpress (and different vendors). The key is knowing and understanding the tradeoffs, then being able to move when the need arises.
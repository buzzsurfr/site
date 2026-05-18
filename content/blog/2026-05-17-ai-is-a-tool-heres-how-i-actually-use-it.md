---
title: "AI Is a Tool. Here's How I Actually Use It."
date: 2026-05-17T19:59:46-04:00
draft: false
categories: ["Deep Dive"]
tags: ["ai", "software", "productivity"]
description: "Every paradigm shift comes with the same two fears. Here's what I've learned about using AI intentionally--from someone who's lived through cloud, DevOps, and containers."
image: ""
---

I remember the cloud conversations. We can't put customer data in the cloud--it's too expensive, and it'll eliminate all the sysadmin jobs. Then came DevOps. Same fears, different vocabulary. Service mesh gave us another round. Now it's AI, and we're right on schedule.

The two fears never change: **it costs too much** and **it will take our jobs**. And they're always at least partially wrong, for the same reason. Both assume passive adoption. The people who got burned by cloud costs weren't the ones using it intentionally. Neither were the sysadmins who got sidelined by automation--they were the ones who refused to learn what automation could do.

## Here We Go Again

The pattern is almost boring at this point:

| Shift | "Too Expensive" | "Takes Jobs" | Reality |
|---|---|---|---|
| Cloud | Egress fees, runaway compute | Sysadmins | Ops roles transformed; cloud skills became table stakes |
| DevOps | Toolchain sprawl, platform costs | Release engineers | Faster delivery, new roles; the resisters fell behind |
| Service Mesh | Proxy overhead, licensing | Network engineers | Absorbed into platform teams; most shops still run it |
| AI | Token costs, subscription sprawl | Developers, writers, analysts | Still playing out--but the pattern holds |

The cost concern is real in each case. You can absolutely light money on fire with AI if you're not paying attention. But the "it takes jobs" fear keeps landing the same way: roles change, they don't disappear. The work shifts. The people who adapt lead the shift. The people who don't adapt eventually get left behind--not by the tool, but by the people wielding it.

## The Accessibility Problem Is Real--But It's Misattributed

Here's the thing the cost critics get right: the low barrier to entry does generate a lot of low-value output.

Someone half-bakes a spec, pastes it into a chat window, ships whatever comes back, and calls it AI development. Someone writes a blog post by typing "write me a blog post about X" and publishes the first result. Someone generates test cases without knowing what the code is supposed to do.

This is what gives AI a bad reputation in serious engineering circles. And the critics aren't wrong that the output is often slop--they're wrong about who's responsible. That's not the tool's fault. That's the user's.

The accessibility that makes AI easy to misuse is the same accessibility that makes it genuinely powerful when you bring discipline to it. A hammer doesn't build bad furniture; bad furniture builders use hammers badly.

## How I Actually Use It

These aren't best practices from a LinkedIn carousel. These are the things I've actually noticed working for me.

**Know what a correct answer looks like before you ask.** This is the one I'd tattoo on every developer who says "I can't tell when AI is hallucinating." You can--if you already know enough about the domain to recognize a wrong answer. If you don't have that baseline, AI isn't your problem. Your baseline is. Use AI to go faster once you know the territory; don't use it to skip learning the territory.

**Be honest about your actual problem.** This sounds obvious. It isn't. Most people sanitize their problem before handing it over--they ask a clean, professional-sounding version of a messy, embarrassing question. AI works best when you bring the real thing. The half-formed thought, the thing you're not sure how to explain, the thing that makes you feel like you should already know the answer. Be vulnerable about what you actually need and the responses get dramatically better.

**Use it as an ADHD co-pilot.** This one's personal, so I'll be direct about it. My brain runs on parallel tracks. I can hold five threads going at the same time and lose the plot on all of them simultaneously. AI holds context across all of those threads without complaint. I can pick up a project mid-thought, dump in where I left off, and it catches up immediately. For a brain that works the way mine does, that's not a nice-to-have--it's the whole value proposition. If this resonates with you, you're not alone, and it's a completely valid reason to use the tool.

**Let it be the second voice when scope explodes.** Every project has that moment where a two-hour task has spiraled into its fifth circle of hell and you're not sure how you got here. AI can notice that before you do. "This has grown beyond the original goal--do you want to pull back to the original scope or adjust the spec?" That question, at the right moment, saves hours. Sometimes I'm the one who asks it. Sometimes I need to hear it.

**Token limits as accidental productivity rhythm.** I didn't plan this one--it happened. Hitting the context limit turned out to be a natural, guilt-free stopping point. No timer. No app. The tool imposed its own rhythm. I'd work hard, hit the wall, and have a clear signal: this session is done. What started as a limitation became a feature. I've since started treating it intentionally rather than waiting for it to happen.

**Intentional constraint is still intentional use.** I'm building a D&D character for a campaign. I've been using AI to work through mechanics--stat allocation, feat synergies, how a particular build interacts with certain encounters. I've been deliberately keeping it out of the plot, the lore, the world-building. My GM controls that narrative and I want to experience it fresh. The tool isn't excluded because it would do a bad job. It's excluded because I made a choice about what I want. That's not avoiding AI--that's using it responsibly.

**Spec-driven development.** This is the one I've written about elsewhere, but it bears repeating here. When I was [building an app from a brain dump](/blog/2026-05-17-from-brain-dump-to-running-app-building-at-the-speed-of-thinking/), every AI change to the codebase prompted the same question: is this also a change to my spec? The spec was the source of truth. The code was just the current expression of it. Keeping them in sync--even by having AI update the spec when it changed the code--kept intention documented. When I hit a dead end with one tool and had to pivot (Retool wasn't the right fit mid-stream), the spec let me pick up cleanly with a different approach. The spec survived the tool change. That's the point.

> The spec is the source of truth; the code is just the current expression of it.

## The Jobs Question, Honestly

Roles change. They don't disappear. That's been true of every shift in this table, and it's going to be true of this one.

The sysadmins who learned cloud became cloud engineers. The release engineers who learned automation became platform engineers. The developers who learn to work with AI will become the ones leading the teams that ship faster than everyone else. The ones who refuse will get left behind--not because AI fired them, but because the person next to them adapted and they didn't.

The real risk isn't that AI takes your job. The real risk is that you decide it won't affect you, someone else decides it will, and you have that conversation with your manager six months later.

## It's Up to Us. It Always Has Been.

Every paradigm shift asks the same question: will you use this, or will it be used on you?

Cloud didn't make infrastructure decisions. Kubernetes didn't decide how your services communicate. AI doesn't decide what's worth building or whether the output is correct or whether you're solving the right problem.

That's still your job. The tool just helps you do it faster--if you're willing to bring the discipline it takes to use it well.

The cost is real. The disruption is real. The opportunity is also real. What you do with all three is the same thing it's always been: up to you.

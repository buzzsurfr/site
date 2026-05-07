---
title: "AI Tooling: Learning the Hard Way"
date: 2026-05-07T10:00:00-04:00
draft: false
categories: ["Project Jumps"]
tags: ["AI", "ollama", "openrouter", "openclaw", "kubernetes", "homelab", "macOS"]
description: "A week of failed experiments with AI tooling — from Kubernetes to a crashed Mac Studio — and what I actually learned along the way."
image: "/images/uploads/ai-tooling-learning-the-hard-way.png"
---

I've been using Claude Desktop and Claude Code for months. They work, and I like them. But I keep wanting *more* — more automation, more local control, more flexibility. (Okay, I fully admit--I want an AI assistant.) That itch led me down a week-long rabbit hole of experiments that went sideways in almost every possible way.

## Attempt One: OpenClaw on Kubernetes

My [homelab runs Kubernetes](/blog/2026-04-03-pibernetes-planning-my-pi-cluster-the-right-way), so naturally my first instinct was to throw [OpenClaw](https://openclaw.ai) on it. That did not go well — not because OpenClaw won't run on Kubernetes, but because I didn't know enough about it to make it work. I was expecting plug-and-play and got a pile of YAML to debug instead. I gave up.

## Attempt Two: Running It Locally ($10 Lesson)

I spun OpenClaw up on my regular machine instead. This required buying Claude API credits (separate from my Claude subscription too). Those credits were gone in an hour -- but the failure taught me something.

I hadn't thought about model selection at all. I was just running whatever was default, and expensive models add up fast. That led me to [OpenRouter](https://openrouter.ai) -- a single endpoint that routes across many models and lets you pick (or auto-select) based on cost and capability. Didn't know that was a thing. Useful.

## Attempt Three: Managed OpenClaw (Too Good to Last)

I found a free-tier managed OpenClaw that actually worked well. I got comfortable with it, started exploring tools and Composio integrations, and was just getting into the interesting parts — when it announced a shutdown. A few days later, it was gone.

Short-lived, but it pointed me toward what I actually want out of this setup. I'll be circling back to tools and Composio once I have a stable foundation.

{{< callout type="warning" >}}
If you run a managed OpenClaw service with a free tier and want a happy early adopter, reach out. You know where to find me.
{{< /callout >}}

## Attempt Four: Vellum

I looked into [Vellum](https://vellum.ai), thinking I was getting something I could run locally or wire up to an open-source model. Instead, it made it very easy to end up on a paid model path, and I burned through the free-tier credits inside an hour. When I tried to switch to a different model, I couldn't figure out how to get back. I just deleted it. (I may owe them $0.28 -- they can try to collect. I'll pay if they also take my feedback and implement it.)

## Attempt Five: Ollama + OpenClaw on My Mac Studio

My Mac Studio has 32 GB of unified memory, so I figured it was a reasonable local host. I grabbed the first model I'd seen recommended — `qwen3-coder` — and fired it up.

That was fun! It immediately consumed all 32 GB and brought my machine to a halt.

So: model size matters. Unified memory is fast, but when the desktop, containers, a 3D printer slicer, and 84 browser tabs are all competing for the same pool -- and then you add a large model on top -- it's a problem. I ended up customizing a model to shrink it down to a size that actually fits in memory. It doesn't perform well yet, but it runs. I've also started learning the difference between model families, quantization levels, and parameter counts — none of which I wanted to care about, but here we are.

The other thing I noticed: running inference locally is *slow*. Uncomfortably slow. I don't yet understand why my hardware isn't keeping up, and that's the next thing I need to figure out.

## Where I Am Now

I still don't have a local AI setup that I'd call working. Things technically run, but performance isn't there yet. The list of open questions looks something like:

- Why is local inference so slow on this hardware, and what can I adjust?
- What model actually fits in memory *and* performs well enough to be useful?
- How do I add tools and Composio to my OpenClaw setup?

The honest truth is that I came into this wanting something that just works. That's not where local AI is right now — at least not for someone coming in without already knowing the ecosystem. I'm clearly late to this game.

For now, I'm back to Claude. I'll keep using it as my daily driver while I continue to experiment on the side. But I'm learning, and I at least know what questions to ask now.

{{< callout type="note" >}}
AI isn't replacing what I do — it augments it. This post is a good example. I came in with a wall of stream-of-consciousness notes and Claude helped me shape them into something readable in under 20 minutes, conversation-style — no task list, no outline, just talking it through. The ideas are mine, the voice is mine — Claude just kept me from rambling. That's the version of AI I actually want. Partially assimilated. Resistance was apparently optional.
{{< /callout >}}

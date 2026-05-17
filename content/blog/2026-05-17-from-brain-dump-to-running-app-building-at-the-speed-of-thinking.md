---
title: "From Brain Dump to Running App: Building at the Speed of Thinking"
date: 2026-05-17T08:00:00-04:00
draft: false
categories: ["Build Log"] # Options: "Build Log", "Deep Dive", "Honest Review", "Project Jumps", "Quick Tip"
tags: []
description: "I built a full-stack character tracker, deployed it to Kubernetes, and shipped a browser extension -- and the part that took the longest was writing the spec."
image: "/images/uploads/from-brain-dump-to-running-app.jpg"
---

I got tired of my D&D characters living in six different places and decided to build something. As it turns out, it took me longer to make that decision than it did to have a working prototype.

## The Retool detour

The first attempt was [Retool](https://retool.com). It's a reasonable choice on paper -- low-code, connects to databases, decent UI builder. I'd used it before for quick internal tools and it worked fine.

Here's the thing about Retool: it's great right up until it isn't. Once I started wiring in anything real, the answer kept being "write some JavaScript in this tiny inline editor." So I started using Retool's built-in AI to write it. And that's when it clicked: if I'm using AI to write code anyway, why am I doing it inside a constrained environment? I'd rather have it write something I actually own and can deploy on [pibernetes](/blog/2026-04-03-pibernetes-planning-my-pi-cluster-the-right-way/). Decision made. Moving on.

## The spec was the real work

I spent more time on the spec than I normally would, for a couple of reasons.

First, I didn't really feel like writing it. Classic.

Second -- and this is the more honest reason -- there were some requirements that needed actual thought. The biggest one: D&D Beyond doesn't have a public API, and that's not changing anytime soon.

D&D Beyond [published their 2026 development roadmap](https://www.dndbeyond.com/posts/2132-d-d-beyonds-2026-development-roadmap#rebuilding-dnd-beyonds-game-platform) earlier this year, and it's genuinely good news. They're rebuilding the backend with a more modular architecture, and for the first time in years it feels like the platform is getting serious investment. Progress is real, and the API is still on the roadmap. I asked about it in the D&D Discord, and the answer I got back was essentially: they want to build it, but infrastructure and retooling the platform has to take precedence first. That's not a dodge -- that's the honest technical reality. You can't publish a reliable API contract against a backend you're actively rearchitecting. I get it.

I'm not waiting.

So the spec needed to account for that uncertainty. Cache as much data locally as possible. Store full raw JSON from the character service. Run a parser to normalize it into structured tables for the items I care about. Track a `source_updated_at` timestamp so the app knows when data has actually changed. The app owns its copy of the data. D&D Beyond is the source, but I'm not dependent on it staying stable.

That kind of thinking only happens before you start building. Getting it right in the spec meant I never had to revisit those decisions once code existed.

## Four commits. Thirty-five minutes.

Once the spec was done, the actual build was fast. Embarrassingly fast.

The first commit dropped the spec at 1:24 PM. By 1:59 PM -- 35 minutes later -- I had the full app: PostgreSQL schema and migrations, all the API routes, a React frontend with character and campaign browsers, and a complete Helm chart with a database for running it on pibernetes. Four commits.

I didn't write any of that code. Claude Code wrote it, and I reviewed it as it came -- understanding it, recognizing the patterns, catching anything that needed adjustment. All of that happened inside those 35 minutes. The difference between this and just telling Claude "make me an app" is that the spec gave it real constraints to work within. The output matched what I actually wanted.

## Then it kept growing

The prototype worked locally. Deploying to the cluster was where things got interesting -- and where I realized this project was going to keep expanding.

Publishing a Docker image to GHCR and pulling it into Kubernetes sounds straightforward. It mostly is, except for the moments it isn't. The database needed its namespace fixed. The credentials weren't wiring up correctly -- the database auto-generates a secret with a `uri` key, and I was referencing the wrong thing. The migration job had a stale `imagePullPolicy` that kept using an old image. None of these were AI failures or human failures in isolation -- that's just collaborative development. The AI generates, you review, you catch things, you iterate. The cycle just runs a lot faster when you're not the one typing everything from scratch.

That's the pattern that kept repeating throughout the week. Problem appears. Fix is obvious once you see it. Move on. The project kept expanding not because I had unlimited time, but because each addition was fast enough that the next thing always felt achievable.

## The parser and why I built a browser extension

Once the app was deployed, I needed to actually get character data into it.

D&D Beyond doesn't expose a public API, but the character sheet page makes API calls to their character service, and those return a rich JSON payload with everything -- classes, subclasses, species, equipment, spell slots, hit points, the works. Other tools like Beyond20 have used this for years. I'm not doing anything novel here; I'm just joining a long tradition of people who needed data that wasn't otherwise available.

Any approach that intercepts those calls automatically is fragile -- D&D Beyond is actively rebuilding their backend right now. I didn't want something that would silently break the next time they shipped an update.

So instead of fighting it, I built a browser extension. When you visit a character page on D&D Beyond, a red d20 shows up in the toolbar. Click it, and it reads the session token from the browser cookies, fetches the full character payload from the character service, and POSTs it to my app's `/api/characters` endpoint as an upsert. A toast tells you if it worked.

Syncing isn't automatic, but it is intentional. I go look at my character sheet, I click the button, the data updates. If D&D Beyond changes their API tomorrow, the extension might break -- but the data I already have is safe in my local database. No silent failures.

It's a single codebase -- the Xcode project for Safari is generated from the same extension folder, which is why it's gitignored. Chrome works today. Firefox support is on my list, but this app isn't going public anytime soon, so that's more of a promise to myself.

## The MCP server

One more thing that ended up in the build: an MCP server.

I already had a D&D Beyond MCP server set up, but it ran into the same fragility problem -- a browser session that needed constant attention to stay alive. Rather than keep wrestling with it, I just added an MCP server directly to the rpg-app that exposes the local database. List characters, get character details, list classes and species. Simple and it works.

It was honestly easier to build one that did exactly what I needed than to keep fixing one that almost did. One prompt. Worked immediately. Still kind of blows my mind.

## Where things are now

The app is running on pibernetes, same as my other services. Characters from a few different campaigns, synced from D&D Beyond via the extension, with VTT links for jumping directly into D&D Beyond Maps or Roll20 during sessions. The campaign view shows everyone in the party, what system we're running, and how to get to the table.

It's not finished -- nothing ever really is. Roll20 and Foundry integrations are on the list, pushing character data directly rather than just linking out.

The real story here isn't D&D Beyond or Kubernetes or browser extensions. It's that I had an idea on a Monday afternoon and was running it in production by Friday -- and the thing that made that possible wasn't coding speed, it was thinking clearly about what I wanted before I started. I came across a piece recently on [how intent is the design superpower that AI actually amplifies](https://somedesigners.substack.com/p/intent-is-the-design-superpower-ai) -- the argument being that people who can clearly articulate what they want will increasingly just build it themselves, and engineering skill gets reserved for the problems that actually require it. That resonated.

This project also left me thinking about how to better organize specs across different projects. Right after I wrapped this one up, a LinkedIn post surfaced a [GitHub article on spec-driven development](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/) and their new open-source toolkit for it. I also came across [OpenSpec](https://openspec.dev), an open-source alternative worth a look. Haven't dug into either yet, but they're on the list.

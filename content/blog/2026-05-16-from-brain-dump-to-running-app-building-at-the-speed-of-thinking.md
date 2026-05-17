---
title: "I Stopped Waiting for the D&D Beyond API"
date: 2026-05-16T12:00:00-04:00
draft: true
categories: ["Build Log"]
tags: ["TTRPG", "Software", "Kubernetes", "D&D"]
description: "I play a lot of TTRPGs across multiple systems and platforms. After years of waiting for D&D Beyond to ship an API, I built my own character tracker and ran it on pibernetes."
image: ""
---

I play a lot of role-playing games. D&D 5e, Pathfinder 2e, DC20, the occasional one-shot in something completely different. The problem is that my characters are spread across D&D Beyond, various PDFs, and whatever the group decided to use for a given campaign. There's no one place to see all of them.

That bothered me enough that I actually did something about it this week.

## The Retool detour

The first attempt was Retool. It's a reasonable choice on paper — low-code, connects to databases, has a decent UI builder. I'd used it before for quick internal tools and it worked fine.

Here's the thing about Retool: it's great right up until it isn't. Getting the basics going was fast, but once I started wiring in anything real — custom queries, slightly non-trivial UI logic — I kept running into walls where the answer was "write some JavaScript in this tiny inline editor." Which is fine, except at that point I was basically writing code anyway, just inside a worse development environment.

At some point I looked at what I was doing and thought: if AI is going to write this either way, I'd rather it write something I actually own and can deploy on [pibernetes](/blog/2026-04-03-pibernetes-planning-my-pi-cluster-the-right-way/). So that's what I did.

## Actually writing a spec this time

I spent more time on the spec than I normally would, for a couple of reasons.

First, I didn't really feel like writing it. Classic.

Second — and this is the more honest reason — there were some requirements that needed actual thought. The biggest one: D&D Beyond doesn't have a public API, and that's not changing anytime soon.

D&D Beyond [published their 2026 development roadmap](https://www.dndbeyond.com/posts/2132-d-d-beyonds-2026-development-roadmap) earlier this year, and it's genuinely good news. They're rebuilding the backend with a more modular architecture, rolling out a better character creation experience, and for the first time in years it feels like the platform is getting serious investment. Progress is real. But a public developer API? Not on the roadmap. It has been requested since the platform launched, went on the roadmap, came off the roadmap, and is now somewhere in the "longer-term" bucket.

I'm among many people who have been asking. I'm done waiting.

So the spec needed to account for that. I wanted to cache as much data locally as possible, because if D&D Beyond changes something under me I don't want to lose everything. The spec called for storing full raw JSON from the character service, running a parser to normalize it into structured tables (classes, species, backgrounds, subclasses), and tracking a `source_updated_at` timestamp so I know when data has actually changed. The app owns its copy of the data. D&D Beyond is just the source.

## Four commits, thirty-five minutes

Once the spec was done, the actual build was fast. Embarrassingly fast.

The first commit dropped the spec into the repo at 1:24 PM. By 1:59 PM — 35 minutes later — I had the full app: PostgreSQL schema and migrations, all the API routes, a React frontend with character browser and campaign browser, and a complete Helm chart with a CloudNativePG cluster for running it on pibernetes. Four commits.

I'm not going to pretend I wrote all of that myself in 35 minutes. Claude Code did the heavy lifting. But the spec was mine, and the decisions in it were real, and watching something go from "here's what I want" to "here's a running thing" in under an hour is still a little surreal.

## Then it kept growing

The prototype worked locally. Deploying it to the cluster was where things got interesting.

Publishing a Docker image to GHCR and pulling it into Kubernetes sounds straightforward. It mostly is, except for the moments when it isn't. The CNPG cluster needed its namespace fixed so it would inherit the Helm release namespace instead of hardcoding one. The database credentials weren't wiring up correctly — CNPG auto-generates a secret with a `uri` key, and I was referencing the wrong thing. The migration job had a stale `imagePullPolicy` that meant it kept using an old image even after I'd pushed fixes. Each of these was a 15-minute debugging session that ended with a small, obvious fix and a slightly better understanding of how the pieces fit together.

This is normal. Kubernetes gives you a lot of power and it's exacting about the details. The cluster is a great place to run things once they're stable; getting to stable takes a lap or two.

## The D&D Beyond parser and why I built a browser extension

Once the app was deployed, I needed to actually get character data into it.

D&D Beyond doesn't expose a public API, but the character sheet page makes API calls to their character service, and those calls return a rich JSON payload with everything — classes, subclasses, species, equipment, spell slots, hit points, the works. Other tools like Beyond20 have used this for years. I'm not doing anything novel here; I'm just joining a long tradition of people who needed data that wasn't otherwise available.

The problem with any approach that relies on intercepting those calls automatically is that the endpoint structure can change. D&D Beyond is actively rebuilding their backend right now. I didn't want to write something that would silently break the next time they shipped an update.

So instead of trying to auto-sync in the background, I built a browser extension. When you visit a character page on D&D Beyond, a red d20 shows up in the toolbar. Click it, and it reads the cobalt session token from the browser cookies (which gives it the same auth as your logged-in session), fetches the full character payload from the character service, and POSTs it to my app's `/api/characters` endpoint as an upsert. A toast notification tells you if it worked.

The nice thing about this model is that syncing is intentional. I go look at my character sheet, I click the button, the data updates. If D&D Beyond changes their API tomorrow, the extension might break, but the data I already have is safe in my local database. I'm not dependent on a continuous background sync that can fail silently.

It works in Chrome, Firefox, and Safari (the Xcode project is gitignored because nobody needs to see that).

## The MCP server

One more thing that ended up in the build: an MCP server.

I already had a D&D Beyond MCP server set up, but it was having issues — it relies on a browser session maintained by Playwright, and that session doesn't survive restarts cleanly. Rather than spend more time wrestling with it, I just added an MCP server directly to the rpg-app that exposes the local database. It surfaces tools for listing characters, getting character details, and listing classes and species. It's simple and it works, which is what I needed.

Having the data in a local database that I own and can query directly is the whole point of this project. The MCP server just makes that data accessible to AI tools in the same way the web UI makes it accessible to me.

## Where things are now

The app is running on pibernetes at `rpg.salvo.services`. It has characters from a few different campaigns, synced from D&D Beyond via the extension, with VTT links for jumping directly into D&D Beyond Maps when we're playing. The campaign view shows everyone in the party, what system we're running, and how to get to the table.

It's not finished — nothing ever really is — but it's useful, which is the bar I care about.

D&D Beyond will probably ship more of that roadmap this year. Maybe eventually an API gets added. When it does, I'll wire this up to it properly. Until then, I've got my one place to see them all.

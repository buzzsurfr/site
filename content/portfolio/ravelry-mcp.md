---
title: "Ravelry MCP Server"
date: 2026-05-12T23:10:16-04:00
draft: false
categories: ["Code"]
tags: ["mcp", "ravelry", "api", "crochet", "claude"]
description: "I built a full-featured MCP server for the Ravelry API using Claude--without writing a single line of code myself."
image: "/images/uploads/ravelry-mcp.png"
project_type: "code"
status: "active"
platform_links:
  - platform: "github"
    url: "https://github.com/buzzsurfr/ravelry-mcp"
  - platform: "npm"
    url: "https://www.npmjs.com/package/ravelry-mcp"
---

As a fiber artist, I live on Ravelry--looking up patterns, tracking my projects, managing my queue, logging my stash. I try to keep it as a reference for myself and as a showcase for my crochet projects.

At the same time, I've been going deep on AI tooling and looking for a real opportunity to build an MCP server--not a toy, something I'd actually use. The connection was obvious: give Claude Desktop direct access to my Ravelry data, so I could ask about my queue or log a finished project without ever opening a browser.

## The Approach

I could have written this myself--TypeScript, REST API, nothing exotic. But I wanted to see how far I could get by letting Claude do the coding while I did the thinking. So far, zero lines written by me.

I fed Claude the Ravelry API documentation and started making decisions out loud: auth model, project structure, which tools to include and which to cut. Claude wrote the code. I reviewed it, caught what broke in production, and directed the next session. Three sessions in, I had something worth shipping.

The result: 21 tools and 5 resources, published on npm as `ravelry-mcp`. You can run it today with `npx ravelry-mcp`.

## What It Does

The server covers the parts of Ravelry I actually use:

- **Pattern search and details**: search by keyword, filter by craft, weight, and category; pull full pattern data
- **Queue management**: view your queue, add patterns, remove entries
- **Project tracking**: list projects, filter by status (in progress, finished, hibernating, frogged), update project details
- **Library and favorites**: browse your saved patterns and stash of saved projects
- **Yarn and stash**: search the yarn database, browse your stash entries

Auth is Basic Auth--two env vars in `claude_desktop_config.json`, one-time setup. No OAuth dance, no hosted relay, no token files.

{{< callout type="note" >}}
I looked at OAuth 2.0 first--Ravelry supports it. But Ravelry requires `https://` redirect URIs, which means a local tool needs a hosted relay just to handle the callback. Once you need hosted infrastructure, the security benefit over a personal API key disappears.
{{< /callout >}}

## What's Next

There's more API surface I haven't touched yet: individual project pages, photo uploads, full stash CRUD, queue reordering, needle inventory, packs. The core use case works well. I'm hoping to get the _fiber artists that also code_ involved to contribute.

## Files & Links

- [GitHub--buzzsurfr/ravelry-mcp](https://github.com/buzzsurfr/ravelry-mcp)
- [npm--ravelry-mcp](https://www.npmjs.com/package/ravelry-mcp)

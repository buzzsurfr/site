---
title: "Finally Getting My Dotfiles Under Control"
date: 2026-04-26T12:00:00-04:00
draft: false
categories: ["Build Log"]
tags: ["Software", "macOS", "Automation", "1Password"]
description: "After years of half-finished dotfiles setups, I finally got it working with a bare git repo, 1Password for secrets, and a lot of AI help."
image: ""
---

I've tried to set up dotfiles properly at least three times. Every time, something doesn't quite click and I abandon it. I recently went on a trip and realized how badly out of sync my machines were--so now I'm committed to getting it right. Let's give AI another spin!

## The Problem With My Previous Attempts

My shell on my travel machine was missing aliases I'd added months ago. Packages weren't installed. The `.zshrc` turned out to be a symlink to a framework I didn't even use anymore. It worked well enough that I never fixed it, until suddenly it mattered.

The root issue wasn't (complete) laziness — it was that I never had a setup that was easy enough to maintain across machines. Every approach I tried either required too much manual work to keep in sync, or broke down when I set up a new machine.

I need a system that automates the tasks in between as well. I want my dotfiles to work without having to do anything day-to-day.

## The Approach: Bare Git Repo

The setup I landed on uses a bare git repo at `~/.dotfiles` with your home directory as the work tree. No symlinks, no extra tools — just git, aliased to `config`:

```bash
alias config="git --git-dir=$HOME/.dotfiles --work-tree=$HOME"
```

Files tracked in the repo live at their real paths (`~/.zshrc`, `~/.ssh/config`, `~/Brewfile`, etc.). Adding a new file is just config add ~/.whatever.

## The Bootstrap Problem

Getting a new machine set up is where dotfile systems usually fall apart for me. The AI suggested an install script at `~/.init/install.sh` handles everything in one shot: clone the repo, install Homebrew, run brew bundle from a tracked Brewfile, set up oh-my-zsh, and configure iTerm2.

## Keeping Secrets Out of the Public Repo

The repo is public, which means anything sensitive has to live elsewhere. Two categories of secrets:

Machine-specific settings — things like iTerm2 window size that differ between machines. Solved with a tracked template (`~/.init/machine.sh.template`) and an untracked local file (`~/.init/machine.local.sh`) that git never sees. The install script reads the local file and applies the settings via PlistBuddy.

Actual secrets — kubeconfigs and SSH keys. These live in 1Password.

For kubeconfig, the setup is a `~/.init/secrets.sh` script that does:

```
op read "op://Private/kubeconfig/notesPlain" > ~/.kube/config
chmod 600 ~/.kube/config
```

For SSH keys, I migrated to 1Password's SSH agent since I run 1Password on all my personal machines and the private keys never need to touch disk at all. The SSH config itself has no secrets in it, so it goes straight into the dotfiles repo.

## Tested on a New Machine

I ran the install script on a new Mac from scratch. The rough order:

1. Run the bootstrap: `bash <(curl -fsSL https://raw.githubusercontent.com/buzzsurfr/dotfiles/main/.init/install.sh)`
1. Sign into 1Password, run `~/.init/secrets.sh`
1. Enable 1Password SSH agent, import keys
1. Copy the machine template and fill in local values
1. It worked. I had to clean up my Brewfile from a few years of bloat--but the fixes went back into the shared repo and now both machines are in sync.

> **Disclaimer:** This setup is tailored for my personal use. You're welcome to copy and adapt it, but you're on your own—use at your own risk!

## What's Next

Still need to backport this to my Linux laptop, which will need some adjustments (no iTerm2, different Homebrew paths, probably some package differences in the Brewfile). But the core approach — bare git repo, secrets in 1Password, idempotent install script — should carry over cleanly.

I also want this to carry over into my Raspberry Pis as well as any codespaces I use in the future. For now, I have my hopes up about this...

---
title: "Pibernetes: Planning My Pi Cluster the Right Way"
date: 2026-03-27T12:00:00-04:00
draft: true
categories: ["Build Log"]
tags: ["Raspberry Pi", "Kubernetes", "Home Lab", "Software"]
description: "I've tried building a Kubernetes Pi cluster before. This time I planned the whole thing before touching a single file — with a little AI help."
image: ""
---

I've tried to build a proper Kubernetes cluster on my Raspberry Pis at least twice before. Both times I got somewhere useful, then hit a wall — some dependency I hadn't planned for, some naming collision, some "I'll figure this out later" that became the thing that killed the whole project.

This time I did something different: I planned the entire thing before writing a single config file.

## The Stack

The cluster is called **Pibernetes**, and it's 10 Pi 4s on a dedicated VLAN with PoE hats supplying power. The naming convention comes from the D&D planes of existence — Sigil sits outside the cluster as the management node (appropriate, since Sigil is the City of Doors at the center of the Great Wheel). The control plane runs on Prime, Feywild, and Shadowfell. Workers are Elysium, Hades, Arcadia, Mechanus, Limbo, and Baator.

The software stack I landed on:

- **k3s** with Flannel, Traefik, and ServiceLB all disabled
- **Cilium** as the CNI — handling pod networking, load balancer IPs (replacing MetalLB), ingress, and Gateway API all in one
- **Tailscale** for external access and subnet routing into the pod and service CIDRs
- **ArgoCD** for GitOps, bootstrapped by Ansible from Sigil
- **Longhorn** for distributed storage on the nodes with direct-attached SSDs
- **cert-manager** with Cloudflare DNS-01 for real TLS on `salvo.services`

The whole thing lives in a private GitHub repo. The plan is to write about the *how* publicly even if the repo stays private.

## How AI Helped With the Planning

I'll be honest about this: I used Claude to work through the architecture with me, and it was genuinely useful in a way I didn't expect.

Not for the answers — I knew most of the answers already. What it was useful for was *forcing the questions*. When I said "just use DHCP," it pointed out that etcd stores peer addresses at bootstrap and a changing IP breaks your control plane. When I said "MetalLB," it surfaced that Cilium ships LB-IPAM natively and I could drop the extra component entirely. When I picked an ingress controller, it walked me through why Cilium's Gateway API support means I can run classic Ingress objects and modern HTTPRoute objects side by side during the transition.

It also just remembered everything. Twenty decisions deep into an architecture conversation, it reflected the full stack back in a clean table without me having to re-explain anything. That part was legitimately useful.

What it couldn't do: make judgment calls about my home network topology, know which planes of existence I'd already used for naming conventions, or decide whether I actually wanted to run PiHole on the Synology or in the cluster. Those still required me.

## What's Actually Planned

The first round isn't about apps — it's about making the platform solid and fully rebuildable. The bootstrap sequence from a freshly flashed SD card to a fully running cluster should be five commands and about 30 minutes. That's the bar.

Platform services going in first:
- Cilium (CNI + LB-IPAM + Ingress + Gateway API)
- ArgoCD
- cert-manager
- Longhorn
- Synology NFS StorageClass
- Prometheus + Grafana
- Tailscale Operator
- PiHole (in-cluster, syncing from the copy on the NAS)

Apps are stubbed in the repo but won't deploy until the platform is solid. When I get there: Forgejo, Plex, Nextcloud, n8n, Minio, and eventually Home Assistant.

## Results

*[Coming soon — build log updates will follow as each layer goes in.]*

---

The repo is private for now, but I'll be writing up each layer here as it comes together. If you're building something similar, the short version of the lesson is: **plan the whole thing before you touch a file, and make sure every layer is independently rebuildable.** The times I've gotten overwhelmed on home lab Kubernetes were always because I built on an unstable layer and didn't know it until three layers later.

More soon.

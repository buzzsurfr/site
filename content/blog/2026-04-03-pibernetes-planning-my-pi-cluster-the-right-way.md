---
title: "Pibernetes: Planning My Pi Cluster the Right Way"
date: 2026-04-03T12:00:00-04:00
draft: false
categories: ["Build Log"]
tags: ["Raspberry Pi", "Kubernetes", "Home Lab", "Software"]
description: "I've tried building a Kubernetes Pi cluster before. This time I planned the whole thing before touching a single file — with a little AI help."
image: "/images/uploads/pibernetes-physical.jpg"
---

I've attempted to build a proper Kubernetes cluster on my Raspberry Pis at least twice before. Both times, I reached a functional state, only to hit a wall—some unplanned dependency, a naming collision, or a "I'll figure this out later" issue that derailed the project into limbo.

This time I did something different: I started with GitOps and the idea that everything be defined via code.

## The Stack

The cluster is called **Pibernetes**, and it's 10+ Raspberry Pi 4s on a dedicated VLAN with PoE hats supplying power. With this iteration, I named my _pets_ using the D&D planes of existence — Sigil sits outside the cluster as the management node (appropriate, since Sigil is the City of Doors at the center of the Great Wheel).

The software stack I landed on:

- **k3s** Traefik and ServiceLB are disabled
- **Flannel** as the CNI. I wanted cilium but there are other issues
- **MetalLB** as the LoadBalancer
- **nginx** as the ingress/gateway
- **Tailscale** for external access and subnet routing into the pod and service CIDRs
- **ArgoCD** for GitOps, bootstrapped by Ansible from Sigil
- **Longhorn** for distributed storage on the nodes with direct-attached SSDs
- **cert-manager** with Cloudflare for DNS/TLS on my domain
- **external-dns** for creating local DNS records with PiHole/CloudFlare
- **prometheus/grafana** for observability

The whole thing lives in a private GitHub repo. While I'm keeping the repo private, I'll share some details here as to how I did it. And in case you get adventurous, I didn't put any secrets into that repo.

## How AI Helped With the Planning

I'll be honest about this: I used Claude to work through the architecture with me, and it was genuinely useful in a way I didn't expect.

Not for the answers — I knew most of the answers already. What it was useful for was *forcing the questions*. When I said "just use DHCP," it pointed out that etcd stores peer addresses at bootstrap and a changing IP breaks your control plane. When I said "MetalLB," it surfaced that Cilium ships LB-IPAM natively and I could drop the extra component entirely (which later also got backed out). It allowed me to have a rational dialogue with...the internet/myself.

For me, with ADHD, I often have 10 conversations at once in my head. AI lets me do that and keeps track of the threads:
- I can hop between “Cilium networking”, “ArgoCD bootstrap”, “DNS-01 certs”, “PiHole placement”, etc.
- It keeps a running context so I don’t lose the intent of each line of thought.
- It’s like having a second brain that can refer back to earlier decisions when I return to a topic hours later.

So yes, I’m not just asking for answers. I’m using it as a stateful planning companion that validates and logs my mental multitasking, which is exactly what makes this workflow workable for me.

What it couldn't do: make judgment calls about my home network topology, know which planes of existence I'd already used for naming conventions, or decide whether I actually wanted to run PiHole on the NAS or in the cluster. Those still required me.

## What's Actually Planned

The first round isn't about apps — it's about making the platform solid and fully rebuildable. The bootstrap sequence from a freshly flashed SD card to a fully running cluster should be five commands and about 30 minutes. That's the bar.

A _fully running cluster_ includes all the platform components above, and with minimal manual steps. In case I decide to nuke everything, I want a process as simple as:

1. Flash SD cards and bring up hosts
1. Run ansible playbook for bootstrap
1. Let argocd install the rest

## Results

So far, the plan is already paying off: I’ve rebuilt the entire cluster twice from scratch in about 25 minutes each time, and both runbooks are now in the repo as executable playbooks. Dropping Cilium for Flannel simplified the networking layer enough that I could stop chasing pod CIDR and service CIDR mismatch issues, and it let me focus on making MetalLB+nginx stable before reintroducing any fancy eBPF-based overlay later.

I did spent close to a week of off-hours getting the configuration the way I wanted it--but it was more of an iterative check than a deep dive. I was able to work on other things while the AI worked on the cluster in the background.

The app list also got a sanity check: PiHole is temporarily paused while I validate core platform metrics/store/backup behavior, and I’m leaning toward a smaller “platform first” set for initial launches (Prometheus/Grafana + Longhorn + cert-manager + ArgoCD) with non-critical workloads following in a second wave.

## Ultimately

My goal is to use this cluster to test other products and ideas from within my own network and push the limits of what's possible from a homelab today. This is the foundation to do so.
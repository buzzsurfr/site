---
title: "Building PI-BERNETES: a home lab"
date: 2022-11-05T11:11:13-05:00
draft: false
categories: ["Honest Review"]
tags: ["homelab", "k3s", "kubernetes", "raspberrypi"]
description: "I bought my first Raspberry Pi (B+) in 2014 when they first launched. I remember buying it because I was spending my time coding but wanted to do so on perso..."
image: ""
---

I bought my first Raspberry Pi (B+) in 2014 when they first launched. I remember buying it because I was spending my time coding but wanted to do so on personal hardware that was accessible and replaceable, and the B+ was $35 USD at the time. I still have it, and it still works (though not in use today).

[![](https://theodorejsalvo.files.wordpress.com/2022/11/rpi1b.png?w=1024)](https://theodorejsalvo.files.wordpress.com/2022/11/rpi1b.png)

At the time of writing, I have 23 different *single board computers (SBC)* but was mostly intrigued by the Raspberry Pi 4 because of the `arm64` architecture and 4 GB available RAM. So I set out to build what was completely unnecessary and yet fun--a Kubernetes cluster out of Raspberry Pies!

## Design Phase

I turned to the one "true" source for inspiration: the internet. [#](https://twitter.com/hashtag/100DaysOfHomeLab)[100DaysOfHomeLab](https://twitter.com/hashtag/100DaysOfHomeLab)

https://twitter.com/Olakipkip/status/1362271263889252352
I really like this case and how clean it looks!

https://twitter.com/rothgar/status/1580626737368825859
A really neat project with some additional ideas on interfacing between the cluster and the environment.

I found a few ideas and started to figure out what my design considerations were.

- **Cable management and airflow is important.** Since I'm an ex-Network Engineer (though those skills have yet to leave me), I wanted to make sure I could keep them running cool without a lot of noise, and that means spending a little extra on power over ethernet (PoE).

- **Modular and expandable.** I've seen the [TuringPi](https://turingpi.com) boards, but this doesn't fit my need as I want to be able to remove or add boards without affecting the surrounding components.

- **Mix of compute and storage.** I knew I had some workloads that would need more than I wanted to (reasonably) fit on a SD card, so I wanted the cluster to support both compute units and storage units. In this case, that's just mounting the hard drives as bays and attaching them to a raspberry pi.

- **Self-sustaining.** I plan to use this cluster for operating my home automation and running private services for projects and community contributions outside of work, so I don't want to depend on any outward services that I can't swap out.

## Hardware

- **Case**

- 1x [C4Labs Cloudlet Case](https://a.co/d/76zJ9ds) - 8-slots with removable mounting plates, includes quiet fans, room for PoE switch, case also mounts to itself (if I go to 16 slots).

- **Compute**

- 4x [Raspberry Pi 4 Model B - 4 GB RAM](https://www.adafruit.com/product/4296) - The base layer

- 4x [LoveRPi PoE Hat](https://a.co/d/gUZv01n) - This has the breakouts for connecting the fans as well as not covering up the chips (so I can add heatsinks)

- [30 pc Heatsinks for Raspberry Pi](https://a.co/d/4Km95vp) - Mix and match, but this had everything I needed to give me a little flexibility with my airflow choices.

- **Storage**

- 2x [Samsung SSD 860 EVO 256GB](https://a.co/d/9oJSOK6) - Fast enough to meet specs on USB 3.0 without costing a fortune.

- 2x [SATA to USB 3.0 cable](https://a.co/d/hiK5byK) - with a short enough lead to fit in with my ethernet cables

- **Network**

- 1x [NETGEAR 8 Port PoE+ Gigabit switch](https://a.co/d/co7JQRw) - capable to push enough power through PoE to also power the hard drive

- 4x CAT6 1 ft Black cable

## Software

**Selecting a container scheduler.** Given my experience with containers, I knew that I wanted to run containers across these devices. With the rise of `arm64` architectures being massively commercialized through [AWS Graviton](https://aws.amazon.com/pm/ec2-graviton/), [Apple silicon](https://www.apple.com/newsroom/2020/06/apple-announces-mac-transition-to-apple-silicon/), [Azure VM](https://azure.microsoft.com/en-us/updates/public-preview-arm64based-azure-vms-can-deliver-up-to-50-better-priceperformance/), and [GCP Tau series compute](https://cloud.google.com/compute/docs/instances/arm-on-compute), I wanted to build an `arm64`-based distro that was capable of running containers. Since I wanted to keep the cluster self-sustained, I ruled out the typical AWS services like ECS Anywhere and EKS Anywhere because they have to communicate with the cloud on some level (plus EKS Anywhere doesn't have arm64 support yet!). Given how much work I do with kubernetes, I wanted to select a k8s distro and ultimately selected [K3s](https://k3s.io) (pronounced "kates") because it's backed by SUSE (Rancher), is lightweight (helps save resources for running containers) and has packaging included.

**Packaging with addons.** Since kubernetes doesn't provide a lot of services on its own (by design), there are a few things to include into this cluster build that will help offer the same services and kubernetes resources like you would get from a cloud-based distribution. K3s includes, helm, serviceLB, and traefik--but it was hard to customize the last two so I disabled them and installed traefik on my own plus MetalLB for load balancing. Since some of the nodes have extra storage, I wanted a storage controller that could integrate with scheduling pods that need hot storage to schedule onto the nodes with the SSDs, and selected longhorn.

Customizing these addons wasn't difficult, but like with many open source solutions, different version documentation can be a real problem. For example, MetalLB recently switched from a ConfigMap to CRDs for defining resources, so it took extra digging to get it running but I did with these resources:

Traefik required customizations, mainly to the helm chart to automatically use the MetalLB load balancer and VIP and to enable ingressClass resources. I also added cert-manager to support encrypted endpoints using LetsEncrypt.

Instead of trying to list every customization, I also spent some time making this process repeatable. I originally bought all this hardware in 2020 and built a cluster but ran into problems early and made too many changes to record. This time, when I started, I made sure I documented the process. My manifests and notes all will end up in a [Github repo](https://github.com/buzzsurfr/homelab) (with the secrets removed) for anyone else to learn from my experiences.

## What's the point?

So far, other professionals would tell you that I have a working kubernetes cluster that does absolutely NOTHING. Why connect all of these nodes together? What can you do with it?

Since I've been an *operator* for most of my career, I tend to get everything ready for use before building a single thing. But I do have ideas of what to run on this cluster and how it's used.

- **Home automation.** I currently have [Home Assistant](https://www.home-assistant.io) running on its own Raspberry Pi (as one of the blades in the picture), but I'd like to move this to containers and work with that community on repeatable processes.

- **Git server.** Sometimes, there are code projects you don't want out on the public internet. I plan to run [Gitea](https://gitea.io/) on this cluster and back it on the SSDs.

- **Home cloud.** If you develop on AWS and haven't seen [LocalStack](https://localstack.cloud), I highly recommend checking it out. The idea started behind `lambda-local` and `dynamodb-local` but quickly expanded and added arm64 support.

- **Minecraft server.** Because I have kids, and one of them is learning to program.

- **Media server.** I have a bunch of DVDs and Blurays that never get used because I'm too lazy to put the DVD in the tray, so I'm gonna digitize them and host on Plex or something similar.

- **Code server.** It's been a dream of mine to work from a tablet, and coding always tends to be one of those misses. At least with [code-server](https://github.com/coder/code-server), I can make it easy to use an IDE (as long as there's reliable internet).

- **Donate unused compute.** There's services like Folding@home and BOINC that allow scientific & academic communities to run their code on remote machines, and I can donate my "unused" CPU cycles to one of these programs. I'll of course prioritize my own workloads, but if I'm not using those cycles then they might as well go to a good cause.

- **Random sparks or ideas.** Because I had set most of this up before KubeCon North America 2022, I had a running cluster ready for running coding challenges and testing out new projects and ideas and was able to complete most of the challenges on the showroom floor, during sessions, or while at the hotel.

Ultimately, having this cluster gives me the freedom to run side projects and test various ideas from my house. It's not *production-ready*, but rather *experimentation-ready*!

[![](https://theodorejsalvo.files.wordpress.com/2022/11/img_4471.jpeg?w=1024)](https://theodorejsalvo.files.wordpress.com/2022/11/img_4471.jpeg)
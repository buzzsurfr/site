---
title: "Match Containers to Host Processes"
date: 2019-02-23T19:51:00-05:00
draft: false
categories: ["Build Log"]
tags: ["docker", "gist", "tmux", "watch"]
description: "During my presentation [Securing Container Workloads on AWS Fargate](https://file+.vscode-resource.vscode-webview.net/Users/salvot/Code/blog/content/blog/sec..."
image: ""
---

During my presentation [Securing Container Workloads on AWS Fargate](https://file+.vscode-resource.vscode-webview.net/Users/salvot/Code/blog/content/blog/securing-container-workloads-on-aws-fargate), I built a demo environment where I could build and run various containers and show the effect they had on the host. While my demo went well, a key piece of feedback is that customers liked how I presented the demo environment by having containers and their host processes on one side. To that end, I'll show you.

### Containers Pane

To show the currently running containers on a given host, use [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/). The normal format (for v18.09.1) looks like:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
525a7b49ef67        nginx               "nginx -g 'daemon of…"   About an hour ago   Up About an hour    80/tcp                   tender_shirley
```

However, for this demo, I was only concerned with the name, image, command, and current status (which has the time it's been running), so I formatted the output using the [`--format`](https://docs.docker.com/engine/reference/commandline/ps/#formatting) flag, and stuck it inside watch to update every second.

##### Command

```
watch -n 1 "docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Command}}\t{{.Status}}'"
```

##### Output

```
Every 1.0s: docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Command}}\t{{.Status}}' localhost.localdomain: Sat Feb 23 13:44:58 2019 NAMES IMAGE COMMAND STATUS tender_shirley nginx "nginx -g 'daemon of " Up About an hour
```

### Host Processes Pane

Getting the host processes (and a way to map them to containers) was more difficult. The best tool in Linux for looking at processes is `ps` (which is where Docker gets the name for `docker ps`), but this doesn't give us *all* the information about a container.

When a container starts, it spawns as a process with a specific process identifier (PID) in the host, but the container sees the PID as **1**. This process can also spawn other processes, which will reference ther parent process PPID. Subprocesses will show up with a PPID of the main process PID but inside the container as PPID 1. For my demo, I wanted to show both the processes and subprocesses at the host, and include information about the user running each process.

Thus, I built a script called [watchpids.sh](https://gist.github.com/buzzsurfr/ad3d29da6324cc290a7ead4270ad38f8). This script gathered the host PIDs, found all of the children PIDs and then fed the list of PIDs to `ps`, also formatting the list to show the running time of the process, the PID, the PPID, the user associated with the process, and the command run. Again, execution of the script was wrapped in `watch`.

##### Script

[gist https://gist.github.com/buzzsurfr/ad3d29da6324cc290a7ead4270ad38f8 /]

With both the containers and processes displayed, map the container `STATUS` to the host process `ELAPSED` time to see what processes show up on the host whenever a new container is started.

### Terminal Window

Tying it all together, I used [`tmux`](https://tmux.github.io/) to build the container and host process panes on the right, and an area to type commands on the left.

`tmux` uses either keyboard shortcuts or commands inside the session to change the environment--going for a "scripting" approach, I chose the latter.

##### Commands

```
tmux new-session -d -s builder_demo
tmux split-window -h
tmux split-window -dv "watch -n 1 \"docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Command}}\t{{.Status}}'\""
tmux select-pane -t 0
tmux send-keys -t 1 'watch -n 1 ./watchpids.sh' C-m
tmux -2 attach-session -d
```

##### Screenshot

[![](https://theodorejsalvo.files.wordpress.com/2021/11/match-containers-to-host-processes-screenshot.png?w=1024)](https://theodorejsalvo.files.wordpress.com/2021/11/match-containers-to-host-processes-screenshot.png)
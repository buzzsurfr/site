---
title: "Rebuilding pi-bernetes over and over again"
date: 2024-01-27T14:18:37-05:00
draft: false
categories: ["Build Log"]
tags: ["ansible", "git", "homelab", "k3s", "kubernetes", "raspberrypi"]
description: "After multiple rebuilds of my homelab cluster and starting a new job, I revisited Ansible to automate the process. I initially crafted a custom playbook for ..."
image: ""
---

While I use my homelab cluster for internal hosting and testing, I also spend significant time fixing and rebuilding it. Since I [first posted about building the cluster](https://theodorejsalvo.com/building-pi-bernetes-a-home-lab/), I've had to stop and rebuild it about 4-5 times. I've made various improvements over time and kept them documented in git, but at some point I don't have a repeatable build for the homelab cluster.

At the same time, my new job has led me to dust off [ansible](https://www.ansible.com) as an operational tool. I've used it in the past (and even ran meetups on it), but I hadn't actually written any playbooks in years. This seemed like a good time to solve both problems at once!

## Reacquainting with ansible & building the playbook

I remembered the syntax and logic of ansible, but there were a few changes since I last used ansible. Fortunately, one of those changes included a [vscode extension](https://marketplace.visualstudio.com/items?itemName=redhat.ansible) for both ansible and a linter! Most of my past playbooks were for F5 and other network devices. Instead of trying to find a device, I just started to build my inventory using the existing pi cluster and *gather facts* about the hosts.

I found the [official k3s-ansible playbook](https://github.com/k3s-io/k3s-ansible) but didn't want to start off using it. Ansible does a good job of abstracting away the mechanics and leaves the end user able to declare their intent--but that's not great for learning. I decided to start from scratch (for now) and create my own playbook based on my current installation with [k3sup](https://github.com/alexellis/k3sup)[1](#cb41c568-b15f-42a4-8594-f65fdb6d59f3). Based on my *many* installations to this same group of hardware, my current installation script looks like:

```
`k3sup install --host azeroth.local \
  --user pi \
  --ssh-key ~/.ssh/pi_cluster \
  --context azeroth \
--cluster \
--local-path ~/.kube/config \
--merge \
--k3s-extra-args '--flannel-backend=wireguard-native --disable=servicelb --disable=traefik' \
--k3s-version=v1.28.2+k3s1
for host (brokenisles eastking kalimdor northrend pandaria)
  do k3sup join --host ${host}.local --server-host 10.20.40.100 --user pi --ssh-key ~/.ssh/pi_cluster --k3s-version=v1.28.2+k3s1
done`
```

With ansible, you need both a playbook (which has plays and tasks) as well as an inventory file. To keep it simple, I wanted an inventory where I just list the hosts and ansible determines who has the control plane role. (Yes, my *theme* this time is World of Warcraft worlds/continents--Lok'tar ogar!).

```
`[k3s]
azeroth
eastking
kalimdor
brokenisles
northrend
pandaria`
```

For the playbook, I used the same strategy and just moved the arguments into a new playbook where I ran the commands based on which host.

```
`- name: K3S control plane
  hosts: k3s[0]
  tasks:
    - name: Install K3S
      ansible.builtin.command:
        argv:
          - k3sup
          - install
          - --host={{ ansible_facts['hostname'] }}.local
          - --user
          - pi
          - --ssh-key
          - ~/.ssh/pi_cluster
          - --context
          - azeroth
          - --cluster
          - --local-path
          - ~/.kube/config
          - --merge
          - --k3s-extra-args
          - '--flannel-backend=wireguard-native --disable=servicelb --disable=traefik'
          - --k3s-version=v1.28.2+k3s1
      delegate_to: localhost
    - name: Record control plane IP
      ansible.builtin.set_fact:
        server_host: "{{ ansible_facts['default_ipv4']['address'] }}"
- name: K3S worker plane
  hosts: k3s[1:]
  tasks:
    - name: Host and IP (debug)
      ansible.builtin.debug:
        msg: "{{ ansible_facts['hostname'] }}: {{ ansible_facts['default_ipv4']['address'] }}"
    - name: Install K3S
      ansible.builtin.command:
        argv:
          - k3sup
          - join
          - --host={{ ansible_facts['hostname'] }}.local
          - --server-host {{ server_host }}
          - --user
          - pi
          - --ssh-key
          - ~/.ssh/pi_cluster
          - --k3s-version=v1.28.2+k3s1
      delegate_to: localhost`
```

Breaking it down, this playbook repeats my custom installation, but wraps it around with ansible. It's not ideal but did give me enough exposure to ansible (again) to move on to my goal: using the [k3s-ansible playbook](https://github.com/k3s-io/k3s-ansible).

## Adding k3s-ansible to the project

After nuking the cluster once again...I was able to clone the project, change my inventory to match the new format, and got the cluster up and running again pretty easily! I then tried moving the playbook into my homelab folder, ran it...and it broke!

I had copied the playbooks, but not the roles, and I had to get the directory structure in proper order. I also knew that by copying files from the project, I'd lose any updates made to the public repo. I wanted to pull updates down, so I instead imported the repo as a submodule and then symlinked the folders I needed to the right spot.

I wanted to hide the submodule(s) (anticipating more for this pattern) and be able to symlink the parts I need from a hidden folder. Thus, I created the folder `.submodules` and added the submodule to that folder.

```
`mkdir .submodules
git submodule add https://github.com/k3s-io/k3s-ansible.git .submodules/k3s-ansible
git submodule init
mkdir playbooks`
```

For the playbooks, I wanted a place where I could pull in the submodule playbooks but also store and create my own. I anticipate needing to add a few things to the cluster immediately after it's built (LoadBalancerClass, CSI, etc.) and I want a singular playbook folder at the root of the project.

```
`mkdir playbooks
ln -s .submodules/k3s-ansible/playbooks playbooks/k3s-cluster`
```

I need the roles to make the playbooks work, but wanted to carry them over individually in case I add roles of my own.

```
`mkdir roles
for role in $(ls .submodules/k3s-ansible/roles/)
do
    ln -s .submodules/k3s-ansible/roles/$role roles/$role
done`
```

I had to redirect the role and inventory lookup to the root of the project. I also enabled caching for my inventory--for what I do, it doesn't hurt.

```
`[defaults]
roles_path = ./roles
inventory  = ./inventory.yaml
fact_caching = jsonfile
fact_caching_connection = ~/.ansible/cache`
```

## A repeatable working cluster

With all this done, I can run `ansible-playbook playbooks/k3s-cluster/site.yaml` and off we go!

```
`PLAY [Cluster prep] ***************************************************************************************

TASK [Gathering Facts] ************************************************************************************
ok: [kalimdor]
ok: [northrend]
ok: [eastking]
ok: [brokenisles]
ok: [azeroth]
ok: [pandaria]

...

PLAY RECAP ************************************************************************************************
azeroth            : ok=31   changed=6    unreachable=0    failed=0    skipped=46   rescued=0    ignored=0
brokenisles        : ok=20   changed=3    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0
eastking           : ok=20   changed=3    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0
kalimdor           : ok=20   changed=3    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0
northrend          : ok=20   changed=3    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0
pandaria           : ok=20   changed=3    unreachable=0    failed=0    skipped=38   rescued=0    ignored=0`
```

There's still work to do. I need to add all the components and operators that I plan to use, and to also put my services back in a reusable (and backed up) format. Stay tuned![2](#7cf3c82d-e2a4-4421-a791-24799b7d0743)
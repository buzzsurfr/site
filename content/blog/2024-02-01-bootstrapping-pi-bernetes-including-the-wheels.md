---
title: "Bootstrapping pi-bernetes: including the wheels"
date: 2024-02-01T07:58:00-05:00
draft: false
categories: ["Build Log"]
tags: ["ansible", "homelab", "kubernetes", "metallb"]
description: "After mastering ansible to automate his homelab Kubernetes cluster, the writer grappled with Gitea installation issues, such as lost configurations without p..."
image: ""
---

In [a previous post](https://theodorejsalvo.com/rebuilding-pi-bernetes-over-and-over-again/), I shared my journey through creating a repeatable build of my homelab cluster using ansible. I can now rebuild Kubernetes anytime I need/want to, but what should I do with it?

## Finding my problem while eating humble pie

One idea is to have a locally-hosted all-in-one git service like Gitea. In previous builds, I started installing gitea using a helm chart. I could then forward the port to my local workstation and I had git!

However, I'm not always at that workstation and need to access gitea without necessarily using `kubectl`, so I opted to create a Load Balancer. K3s does include ServiceLB, but it lacks features and didn't work out of the box on my network. MetalLB has the support and community, so I went and grabbed that helm chart and installed it. Presto! Now I can support load balancers.

Then, I had to restart a pod--and lost my gitea installation. I didn't enable persistent storage on my gitea deployment. Well to do that, I need to check the CSI drivers. There's the default `local-path`, but that doesn't allow my pods to move. Since Rancher makes both K3s and Longhorn, I fetched the longhorn helm chart and had persistent storage.

Then I needed to customize Traefik (installed by default) and broke it...

...and wanted to monitor everything, so put prometheus on, and broke it again...

..and there came a point where I questioned whether I was really experienced at Kubernetes at all![1](#4d986787-d68d-495a-a392-77dcf063f2b5)

**My problem wasn't experience or knowledge based, but rather *how*** I had chosen to operate. Every time I rebuilt the cluster, I would say to myself *"I should probably automate this--I'll do it after I build it"*...and never go back to it.

I realized that most of my IT career had been spent watching customers and clients install a package to a linux server, or build a new S3 bucket in the AWS console, or apply a schema patch to a database...

...and I had just done the same thing!

My proposed solution had always been the same: *just automate it*. So I did.

Now, I can completely wipe off k3s from the SBDs, and in one command get it running again.

## Attaching the wheels to the frame

With a Kubernetes cluster, I have a frame(work) that I can put widgets on. Like a car can't go anywhere without wheels (still waiting for my flying car, thanks [Back to the Future Part II](https://www.imdb.com/title/tt0096874/)), my Kubernetes cluster needs some support before I can use it for my true goals. I need MetalLB, a CSI, a customized traefik, etc.

One reason I picked ansible for building the cluster was that I could use it to both deploy the cluster AND the Kubernetes resources. I also considered OpenTofu (not Terraform--[here's why](https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license)) and had a few other suggestions (which I haven't really looked at yet). I may go that direction in the future, but borrowing the leadership principle *Bias for Action*, I picked one and can always change it later.

> 

**Bias for Action**
Speed matters in business. Many decisions and actions are reversible and do not need extensive study. We value calculated risk taking.

-Amazon Leadership Principles

I started with a basic playbook template to make sure I could query Kubernetes by listing the namespaces in the cluster.

```
`---
- name: Kubernetes Components
  hosts: kubernetes
  gather_facts: false
  tasks:
    - kubernetes.core.k8s_info:
        context: k3s-ansible
        kind: Namespace
      register: ns
    - ansible.builtin.debug:
        var: ns.resources | map(attribute='metadata.name') | list`
```

I have this host entry in my `inventory.yaml` file as well. This lets me specify `kubernetes` as the host above.

```
`kubernetes:
  hosts:
    k8s-azeroth:
  vars:
    ansible_connection: local
    ansible_python_interpreter: "{{ansible_playbook_python}}"`
```

As a quick test, I get this output.

```
`PLAY [Kubernetes Components] ******************************************************************************

TASK [kubernetes.core.k8s_info] ***************************************************************************
ok: [k8s-azeroth]

TASK [ansible.builtin.debug] **#***************************************************************************
ok: [k8s-azeroth] => {
    "ns.resources | map(attribute='metadata.name') | list": [
        "kube-system",
        "kube-public",
        "kube-node-lease",
        "default"
    ]
}

PLAY RECAP ************************************************************************************************
k8s-azeroth        : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   `
```

I now have an easy mechanism to call the kubernetes API from within my same ansible structure!

## Adding the first component - MetalLB

After looking at the MetalLB installation guide, it also supports kustomize, so I tried to setup kustomize through ansible. The task is still `kubernetes.core.k8s`, but there's a lookup module specifically for kustomize). The task looks like this:

```
`    - name: Network - MetalLB
      kubernetes.core.k8s:
        state: present
        namespace[2](#d73d7f86-775b-45dc-adb3-33401d26d347): metallb-system
        definition: "{{ lookup('kubernetes.core.kustomize', dir='github.com/metallb/metallb/config/native?ref=v0.13.12' ) }}"
      tags: network`
```

It took some investigation, but the task above is the equivalent of this `kubectl` command:

```
`kubectl create -n metallb-system -k github.com/metallb/metallb/config/native?ref=v0.13.12`
```

Each task supports tags, which I can use later to only install a certain type of component. In this case, I could limit the tasks to the network tag. While it's not necessary now, it becomes useful very fast.

MetalLB also takes a little extra configuration, which is provided in the form of CustomResources. In my homelab, I have carved out a specific IP range for the load balancer, and I assign it to this cluster with this task:

```
`    - name: Network - LoadBalancer IP addresses
      kubernetes.core.k8s:
        state: present
        src: ../manifests/metallb/ipaddresspool.yaml
      tags: network`
```

For reference, `ipaddresspool.yaml` contains:

```
`---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 10.20.40.10-10.20.40.99
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system`
```

Alternatively, I can use the full power of the [kubernetes.core.k8s](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html) module to rearrange and pull files or definitions as necessary. For example, I could combine both files into two ansible tasks, placing the resource definition *verbatim* under the **definition:** property.

```
`    - name: IPAddressPool
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: metallb.io/v1beta1
          kind: IPAddressPool
          metadata:
            name: default
            namespace: metallb-system
          spec:
            addresses:
            - 10.20.40.10-10.20.40.99
      tags: network
    - name: L2Advertisement
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: metallb.io/v1beta1
          kind: L2Advertisement
          metadata:
            name: default
            namespace: metallb-system
      tags: network`
```

This is the flexibility I was looking for, and I'm using the same tool for everything thus far!

## Foreward

I'm documenting my complete stack ("eventually"), but I can use this pattern to add different tasks and plays in the same way I'd manage helm charts, resource definitions, or kustomizations. I'd like to try the same setup with terraform or other tools (or to read someone else's blog about it!), but first I have more components to install before I can put Gitea on my cluster!
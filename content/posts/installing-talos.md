+++ 
draft = true
date = 2026-08-02T14:21:21+02:00
title = "How i'm setting up my kubernetes homelab"
description = "A walkthrough of how i get my homelab setup with talos and fluxcd"
slug = ""
authors = ["Bjørn Kristian Strand"]
tags = ["talos", "flux", "kubernetes"]
categories = ["Homelab"]
externalLink = ""
series = []
+++

After having used proxmox on my homelab for a couple of years and virtualizing all my workloads (including my kubernetes clusters) i am now moving to bare metal kubernetes for a couple of reasons. 1) I have no interrest in maintaining a virtualization layer in my homelab. and 2) I'm moving over from k3s to talos, so my kubernetes setup is getting overhauled either way.

K3s is a great technology for running kubernetes with a low footprint, but for the same reason i wanted to move away from proxmox, i wanted to move away from k3s because i really do not care about maintaining the OS of my kubernetes node(s). Granted, there is a certain level of "OS-management" you have to do on Talos aswell, but the scope is considerably smaller and is reasonably easy to perform using `talosctl` instead of spending a lot of time writing ansible-playbooks.

All the code for this cluster is located at [bksuup/quasar](https://github.com/bksuup/quasar)

## General overview of the setup

As mentioned, this new setup will be using [Talos](https://docs.siderolabs.com/talos/v1.13/overview/what-is-talos) as the kubernetes operating-system, and i'll be using [FluxCD](https://fluxcd.io/flux/) as my gitops solution. Other notible choices of software wich will be covered in this post is: [Cilium](https://docs.cilium.io/en/stable/) as my CNI and [Longhorn](https://longhorn.io/docs/1.12.0/) for block storage.

This new setup is running Talos on just one physical node for a couple of reasons.
1. This cluster is not "production" in the sense that i have no fear of loosing any data hosted in the cluster. This clusters entire purpose is to function as a lab-environment.
2. As mentioned, i'm not interessted in managing a virtualization-layer so that i can run my kubernetes-cluster in proxmox-VM's. There is a certain level of management that i don't want to do, and i want the most ammount of "raw compute" that i can get going to my cluster instead of being lost through virtualization.
3. Hardware is expensive, and for a lab-environment, the benefits of having two more physical nodes does not justify the cost.

So, in short: one bare-metal host running talos-linux, using Cilium as a CNI, fluxcd for gitops, and longhorn for block-storage. Quite simple.

## Install prerequisites

### Tools

There are a few thing we have to consider when installing this cluster.
Firstly, i'll be doing this manually.
There are several ways to automate the install process with tools like terraform and ansible, but for me, this is a one-time setup where the focus is not on "automating the install", but just "get a cluster up and running", so automating it isn't something i want to spent time on (though you will learn a lot from it if you do).
For the manual install process you need to install a few cli tools:
- [flux-cli](https://fluxcd.io/flux/cmd/)
- [talosctl](https://docs.siderolabs.com/talos/v1.13/getting-started/talosctl)
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [helm](https://helm.sh/docs/intro/install/)
- [cilium cli](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli) (optional, not needed for the install)
- [k9s](https://k9scli.io/topics/install/) (optional, but extremely nice2have)

### Talos system extensions

Since i'll be using Longhorn as my block-storage provider, i have to download the Talos ISO image with a couple of system extensions installed (you can read more at [longhorn#prerequisites](https://docs.siderolabs.com/kubernetes-guides/csi/longhorn)). These include `siderolabs/iscsi-tools` and `siderolabs/util-linux-tools`.
These are the only two system extensions needed for this install.
> Since Talos is an immutable linux distro, you cannot "just install" the extensions in the running cluster. The extensions are baked into the image, so if you need to install new extensions down the line, you will have to download a new image with all the extensions installed and patch the cluster with the new iso image.

### Repository setup

I will be following [this guide](https://github.com/fluxcd/flux2-kustomize-helm-example) from fluxcd on how to structure your repository. The final structure looks like this.

```markdown
quasar
├── apps
│   ├── base
│   └── prod
├── clusters
│   └── prod
└── infrastructure
    ├── base
    └── prod
```

This allows me to add a staging cluster down the line without having to change the structure of the repo, or re-write deployments.

### Talos Machine Config

There are a couple of settings i'll have to set when creating the machine config for talos, i'll list them here along with the documentation if you're interested to read more, but they're pretty self-explanatory.

```yaml
# prerequisite for cilium install
# https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium#machine-configuration-prerequisites
cluster:
  network:
    cni:
      name: none
  proxy:
    disabled: true
```

```yaml
# Allow scheduling of workloads on control plane nodes since it is a single node cluster
# https://docs.siderolabs.com/talos/v1.13/deploy-and-manage-workloads/workloads-on-controlplane
cluster:
  allowSchedulingOnControlPlanes: true
```

```yaml
# pre-requisite for metrics-server
# https://docs.siderolabs.com/kubernetes-guides/monitoring-and-observability/deploy-metrics-server
machine:
  kubelet:
    extraArgs:
      rotate-server-certificates: true
```

Additionally, when first booting talos on my home-network, the machine will get a DHCP-assigned IP which will change when the machine reboots. This following talos patch is used to set static ip and other network settings.
You can find the interface name by running the following `talosctl` command:
```sh
talosctl -n <node-ip> get links
```

```yaml
machine:
  network:
    interfaces:
      - interface: <interface-name> # use your actual interface name
        dhcp: false
        addresses:
          - <static-ip>/<network-mask>
        routes:
          - network: 0.0.0.0/0 # Default route
            gateway: <gateway-ip>
    nameservers:
      - <ns-ip>
```

+++ 
draft = false
date = 2026-07-06T16:12:52+02:00
title = "Bootstrapping my new bare-metal kubernetes cluster with talos"
description = "How i built my new bare-metal kubernetes cluster running talos"
slug = ""
authors = ['Bjørn Kristian Strand']
tags = ['homelab', 'kubernetes']
categories = []
externalLink = ""
series = []
+++

After using [proxmox](https://www.proxmox.com/en/) on my homelab for a few years and virtualizing my workloads (both kubernetes and regular VMs), i decided that i wanted to 1) remove the virtualization overhead as i was just wasting resources on a virtualization-layer i did not want to maintain, and 2) make the switch from [k3s](https://k3s.io/) to [Talos](https://www.siderolabs.com/talos-linux) for my kubernetes clusters (calling it a "cluster" is a stretch as i currently just have one node).

K3s is a great technology for running kubernetes with a low footprint, but for the same reason i wanted to move away from proxmox, i wanted to move away from k3s because i really do not care about maintaining the OS of my kubernetes node(s). Granted, there is a certain level of "OS-management" you have to do on Talos aswell, but the scope is considerably smaller and is reasonably easy to perform using `talosctl` instead of spending a lot of time writing ansible-playbooks.

This post won't go too much into detail on the physical setup (mostly because it's not really that interesting), but in short, i have a single physical machine with 8 cores (16 threads), 32gb of DDR4 RAM, 1tb NVME-ssd, and has a single nic that plugs into my home-router. This is more than enough for running some homelab workloads on kubernetes.

All the code for this cluster is located at [bk-homelab/quasar](https://github.com/bksuup/quasar)

## Installing Talos

I'll be following the official talos [Getting Started](https://docs.siderolabs.com/talos/v1.13/getting-started/getting-started) guide, and the steps are generally the same as a normal install of talos: install `talosctl`, download the talos ISO, and boot into the ISO such that the machine enters maintenance-mode.

### Generating the cluster config

At step 6 in the talos install guide you have to generate a cluster configuration which is used in the bootstrap command that actually installs talos onto the machine. I have a couple patches located at `talos/` in my git-repository which are pre-requisites for later setup.

The patches are as follows

``` yaml
# control-plane-scheduling.yaml
# Allow scheduling of workloads on control plane nodes since it is a single node cluster
cluster:
  allowSchedulingOnControlPlanes: true
```

``` yaml
# cilium-patch.yaml
# prerequisite for cilium install
cluster:
  network:
    cni:
      name: none
  proxy:
    disabled: true
```

``` yaml
# kubelet-cert-rotation.yaml
# pre-requisite for metrics-server
machine:
  kubelet:
    extraArgs:
      rotate-server-certificates: true

```

We can now generate the config using the following command

``` shell
talosctl gen config $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 \
  --install-disk /dev/$DISK_NAME \
  --config-patch @talos/control-plane-scheduling.yaml \
  --config-patch @talos/cilium-patch.yaml \
  --config-patch @talos/kubelet-cert-rotation.yaml 
```

If you are planning on using this blog-post as a guide for your own install, know that after you apply the machine configuration with CNI set to `none` in the next step as i have, you have 10 minutes to install a CNI (in my case cilium) before the node reboots on its own as the install is not successfull until you have a CNI up and running. Read the next section which details the install of cilium and do the pre-requisites so that you can easily install the CNI before the 10min timer.

Apply the config and initate bootstrapping talos onto the node.

``` shell
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
```

Follow the rest of the steps from the talos install guide which gives you access to the cluster (both talos and kubernetes api's) and bootstraps ETCD.

## Installing cilium (Container Network Interface)

For the initial install we will be installing cilium using the cilium cli as this is the quickest way to get a CNI up and running. After we have configured the CD solution, cilium will also be managed via gitops.

Follow the [official documentation](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/) from cilium to install the cilium cli.

Once the talos host is done with applying the machine configuration, start the installation of cilium onto the cluster. Talos has its own [documentation for installing cilium](https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium#method-3-cilium-cli) as there are a few install-options we have to consider.

``` shell
cilium install \
    --set ipam.mode=kubernetes \
    --set kubeProxyReplacement=false \
    --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
    --set cgroup.autoMount.enabled=false \
    --set cgroup.hostRoot=/sys/fs/cgroup
```

After starting the installation, you can run `cilium status --wait` which will live-refresh the status of cilium. After a while cilium will be done installing and you will have an output that looks like this.

``` shell
$ cilium status
    /¯¯\
 /¯¯\__/¯¯\    Cilium:             OK
 \__/¯¯\__/    Operator:           OK
 /¯¯\__/¯¯\    Envoy DaemonSet:    OK
 \__/¯¯\__/    Hubble Relay:       disabled
    \__/       ClusterMesh:        disabled

DaemonSet              cilium                   Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet              cilium-envoy             Desired: 1, Ready: 1/1, Available: 1/1
Deployment             cilium-operator          Desired: 1, Ready: 1/1, Available: 1/1
Containers:            cilium                   Running: 1
                       cilium-envoy             Running: 1
                       cilium-operator          Running: 1
```

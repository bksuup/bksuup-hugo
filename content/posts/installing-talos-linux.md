+++ 
draft = true
date = 2026-07-06T16:12:52+02:00
title = "Installing bare-metal Talos Linux in my homelab"
description = "How i install and configure talos linux on my bare-metal homelab server"
slug = ""
authors = ['Bjørn Kristian Strand']
tags = ['homelab', 'kubernetes']
categories = []
externalLink = ""
series = []
+++

After using [proxmox](https://www.proxmox.com/en/) on my homelab for a few years and virtualizing my workloads (both kubernetes and regular VMs), i decided that i wanted to 1) remove the virtualization overhead as i was just wasting resources on a virtualization-layer i did not want to maintain, and 2) make the switch from [k3s](https://k3s.io/) to [Talos](https://www.siderolabs.com/talos-linux) for my kubernetes clusters (calling it a "cluster" is a stretch as i currently just have one node).

K3s is a great technology for running kubernetes with a low footprint, but for the same reason i wanted to move away from proxmox, i wanted to move away from k3s because i really do not care about maintaining the OS of my kubernetes node(s). Granted, there is a certain level of "OS-management" you have to do on Talos aswell, but the scope is considerably smaller and reasonably easy to perform using `taloscli` instead of spending a lot of time writing ansible-playbooks.

This post won't go too much into detail on the physical setup (mostly because it's not really that interesting), but in short, i have a single physical machine with 8 cores (16 threads), 32gb of DDR4 RAM, 1tb NVME-ssd, and has a single nic that plugs into my home-router. This is more than enough for running some homelab workloads on kubernetes.

## Preparing for install

<add install of talosctl>

I'll be following the official talos [Getting Started](https://docs.siderolabs.com/talos/v1.13/getting-started/getting-started) guide up to step 6.

This blog-post assumes you have basic knowledge of installing linux onto a physical machine, and while talos deviates from the "regular" install experience, how you create the physical boot-device is the same.
Go to Talos [Image Factory](https://factory.talos.dev/) and chose the options that fits your system / requirements. For my setup this is `Bare-metal machine`, `The latest version of talos`, `amd64`, no system extensions, and `bootloader: auto`. This will give you an iso you can download to your local machine.

Depending on the platform you're preparing your iso from, you can use either [Rufus(windows)](https://rufus.ie/en/) or dd(linux). Insert the usb-drive (at least 8gb) and write the ISO to the usb.
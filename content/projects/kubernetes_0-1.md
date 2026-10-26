+++ 
draft = true
date = '2026-10-26T12:38:34+01:00'
title = 'Kubernetes: from 0 to 1'
description = "How i deploy a functional kubernetes cluster on bare metal"
slug = ""
authors = ['Bjørn Kristian Strand']
tags = ['homelab', 'kubernetes']
categories = []
externalLink = ""
series = []
+++

This project will be a detailed walkthrough of how i have configured my kubernetes cluster running on a bare-metal server, including how the physical nodes are set up, what software i will be using for the cluster, how my gitops solution works and how i have constructed my workflow around this cluster.

This initial post will be covering how to get to a functional cluster.
There are several additional services that need to be configured in order to serve as a development platform for application development, but that will be covered in another post.

The source-code for my cluster is unfortunately not publicly available, but i will be covering the structure of the repository, along with concrete code-examples, meaning that if you wanted to you could create an equivalent infrastructure yourself.

## The physical node

- What hardware the node has
- the disk-configuration

## Choosing a kubernetes solution

- Why talos

## Configuring Talos

- Image factory (Justfile)
    - necessary extensions
- Machine config (Justfile)
- Bootstrapping the cluster
- Installing cilium
- metrics-server and CSR approver

## Configuring GitOps

- repository structure (flux example repo)
- Adding Sops encryption
- Bootstrapping fluxcd

## Exposing services with TLS encryption

- Envoy + Cert Manager

## Storage on a single-node "cluster"

- Longhorn-configuration

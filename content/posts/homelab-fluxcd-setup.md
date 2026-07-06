+++ 
draft = true
date = 2026-07-06T13:00:52+02:00
title = "Configuring FluxCD for my kubernetes homelab"
description = "How i install and configure FluxCD for my homelab running kubernetes on Talos"
slug = ""
authors = ['Bjørn Kristian Strand']
tags = ['fluxcd', 'kubernetes', 'homelab', 'gitops']
categories = []
externalLink = ""
series = []
+++

One of the first problems you have to address when building a kubernetes-cluster for your homelab is to pick which CD-tool you are going to base your gitops workflow on.
The two most common candidates for this is ArgoCD and FluxCD, both great choices that are widely used accross the industry and have their own advantages and drawbacks.
In this post i'll talk about why i went with Flux instead of Argo, how i have implemented Flux in my current homelab setup, and <insert reason here>

## Chosing FluxCD over ArgoCD

## Repository structure

## Bootstrapping Flux
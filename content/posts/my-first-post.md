+++ 
draft = true
date = 2026-03-29T18:02:43+02:00
title = "Introduction"
description = ""
slug = ""
authors = []
tags = ["test", "testing"]
categories = []
externalLink = ""
series = []
+++

## Introduction

This is **bold** text, and this is *emphasized* text.

Visit the [Hugo](https://gohugo.io) website!

## testing

```sh
# this is a test command to check syntax-highlighting
echo "test" > file.txt
```

## Golang

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {

    f, _ := strconv.ParseFloat("1.234", 64)
    fmt.Println(f)

    i, _ := strconv.ParseInt("123", 0, 64)
    fmt.Println(i)

    d, _ := strconv.ParseInt("0x1c8", 0, 64)
    fmt.Println(d)

    u, _ := strconv.ParseUint("789", 0, 64)
    fmt.Println(u)

    k, _ := strconv.Atoi("135")
    fmt.Println(k)

    _, e := strconv.Atoi("wat")
    fmt.Println(e)
}
```

## Yaml

```yaml
---
services:
  traefik:
    image: traefik:v3.4
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true

    networks:
      - frontend

    ports:
      - '${FRONTEND_IP}:80:80'
      - '${FRONTEND_IP}:443:443'
      - '${FRONTEND_IP}:8080:8080'

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./certs:/certs:ro

    environment:
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}

    command:
      # Entrypoints
      - '--entrypoints.web.address=:80'
      - '--entrypoints.web.http.redirections.entrypoint.to=websecure'
      - '--entrypoints.web.http.redirections.entrypoint.scheme=https'
      - '--entrypoints.web.http.redirections.entrypoint.permanent=true'
      - '--entrypoints.websecure.address=:443'
      - '--entrypoints.websecure.http.tls=true'

      # Providers
      - '--providers.docker=true'
      - '--providers.docker.exposedbydefault=false'
      - '--providers.docker.network=frontend'

      # Let's Encrypt provider
      - '--certificatesresolvers.le.acme.email=${ACME_EMAIL}'
      - '--certificatesresolvers.le.acme.storage=acme.json'
      - '--certificatesresolvers.le.acme.caServer=https://acme-v02.api.letsencrypt.org/directory'
      - '--certificatesresolvers.le.acme.keyType=EC256'
      - '--certificatesresolvers.le.acme.dnsChallenge.provider=cloudflare'
      - '--certificatesresolvers.le.acme.dnsChallenge.resolvers=${DNS_RESOLVERS}'

      # API & Dashboard
      - '--api.dashboard=true'
      - '--api.insecure=false'

      # Observability
      - '--log.level=INFO'
      - '--accesslog=true'
      - '--metrics.prometheus=true'

    labels:
      # Enable self‑routing
      - 'traefik.enable=true'

      # Dashboard router
      - 'traefik.http.routers.dashboard.rule=Host(`${DASHBOARD_URL}`)'
      - 'traefik.http.routers.dashboard.entrypoints=websecure'
      - 'traefik.http.routers.dashboard.service=api@internal'
      - 'traefik.http.routers.dashboard.tls=true'
      - 'traefik.http.routers.dashboard.tls.certresolver=le'

      # Basic‑auth middleware
      - 'traefik.http.middlewares.dashboard-auth.basicauth.users=${DASHBOARD_AUTH_TOKEN}'
      - 'traefik.http.routers.dashboard.middlewares=dashboard-auth@docker'

networks:
  frontend:
    external: true
```

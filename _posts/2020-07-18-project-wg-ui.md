---
layout: post
title: About project wg-ui
date: 2020-07-18 23:50
category: project
author: m0sscode
tags: [golang, wg]
summary:
---

## Backend

At first, we assume known how to use `WireGuard` already. Let's start at the backend.
The backend is implemented by Golang. The code files' structure look like this.

```bash
.
├── config.go
├── main.go
├── server.go
```

It's pretty simple. `main.go` is the entrypoint to instruct. `config.go` and `server.go` implement service logic.

At lines 33~52, some global variables are defined and init from the command flags.
Some of them are used to config the server's entry point(`listenAddr`) or persistence folder(`dataDir`).
Some of them are used to config WireGuard server. `wgLinkName`, `wgListenPort`...
Others are used to generate user config and another user. `clientIPRange`, `authUserHeader`.

The usage of `devUIServer` is interesting. I always configure frontend project proxy at the develop stage.
Never thought about adding a dev proxy on the backend.

What `NewServer` func does is prepare config and embedded static file of the frontend.
The embedding tool it uses looks like not maintained.

`Start` func will apply the config to a network device, serve static and APIs.

`initInterface` impl the instruction to init and setup a wireguard peer with the config predefined.

The backend APIs are about CRUD client config.

`CreateClient` will create a pair key and update the config file with a lock.

## Frontend

Frontend uses Svelte to develop. By researches, I have known Svelte is more suitable for small projects than Vue and React. Because the most function of framework is based on operating DOM directly, not through virtual DOM.

Four major components are `EditClient`, `NewClient`, `About`, `Clients`. The design of the project is simple and clear.

## Issues



- `Errors should give more context`. Running in docker and errors for "operation not supported". This maybe relates to network cap or userspace wireguard. Need to reproduce on Debian.
- `Possibility to hide or remove a "newClient" button`. Control `newClient` button base on the number of clients. Need to familiar Svelte. Maybe need add API.
- `Kubernetes`. Similar to the first one. Experiments with isolated and non-isolated networks could be arranged.
- `ipv6 support`. Need to familiar config `ipv6`.

If available later, take a look running in docker with different network isolation on the specific os and kernel.

To be continued.
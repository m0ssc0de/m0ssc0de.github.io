---
layout: post
title: Start a web project with Rust(1/2)(unfinished)
date: 2020-05-18 08:41
category: 
author: m0ssc0de
tags: [rust, web, boot]
summary: 
---

In the past weeks, I started a project with Rust. Web frontend and web backend both were written by Rust. A kubernetes were used as a functional backend by managing CRD resources. And now let discuss them one by one.

First part is about web layer. The frontend framework use `yew`. CSS use `uikit`. The web backend server use `actix`. Use `diesel` to manage database operating. `prost` is used to manage data serialization. The communication betweent frontend and backend use `websocket` managed by `actix`.


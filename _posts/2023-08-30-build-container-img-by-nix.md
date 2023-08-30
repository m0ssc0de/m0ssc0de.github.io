---
layout: post
title:  "Getting Docker Images in a More Declarative Way"
date:   2023-08-30 16:55:07 +1200
category: 
author: m0ssc0de
tags: [nix, docker, image, k8s, kubernetes]
summary:
---
# Getting Docker Images in a More Declarative Way

This post serves as a sequel to my earlier article, [How Many Ways to Get a Docker Image?](https://m0ssc0de.github.io/docker/image/distribution/2020/04/29/how-many-ways-to-get-a-docker-image.html).

## Introduction
The inspiration for this post came from pondering how to integrate Nix with Kubernetes to enhance efficiency and reduce costs. After brainstorming, I concluded that Kubernetes already handles networking and storage well. However, there's room for improvement in how it manages execution environment. This post explores that avenue.

## Two New Approaches
I'll introduce two new methods for building Docker images:

1. **Declarative Image Building**: Unlike traditional Dockerfiles, this approach doesn't rely on step-by-step instructions. Instead, it references the libraries that the image will require.
  
2. **Universal Docker Images**: This method can be extended to create a universal way to build Linux distribution images. Stay tuned for a future post on this topic.

### Existing Methods for Reference

[How Many Ways to Get a Docker Image?](https://m0ssc0de.github.io/docker/image/distribution/2020/04/29/how-many-ways-to-get-a-docker-image.html).

- `docker pull`
- `docker commit`
- `docker build`
- `docker load`
- `docker import`
  - `yum` for CentOS
  - `debootstrap` for Debian
  - `squashfs` for any

## Declarative Image Building with Nix
Let's assume you need an environment to interact with Redis, Kubernetes API server, Google Cloud services, and a JSON-producing HTTP API. Normally, you'd write a Dockerfile with installation instructions. However, using the Nix file below ensures that you get identical Docker images with ***EXACTLY SAME HASH*** every time you build.

```shell
$ cat hello-docker.nix 
{ pkgs ? import <nixpkgs> {} }:

pkgs.dockerTools.buildImage {
  name = "my-image";
  tag = "latest";
  copyToRoot = [
    pkgs.google-cloud-sdk
    pkgs.redis
    pkgs.kubectl
    pkgs.curl
    pkgs.jq
    pkgs.bashInteractive
  ];
  config = {
    Cmd = [ "/bin/bash" ];
  };
}
$ nix-build hello-docker.nix 
/nix/store/zicgvkjp6rgqa7v9z89r58hgxf11mxmj-docker-image-my-image.tar.gz
$ docker load < result 
Loaded image: my-image:latest
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
my-image     latest    db6bf36eb8ed   53 years ago   1.42GB
```

This approach guarantees reproducible builds, which is essential for CI/CD pipelines that aim for more than just script execution.

## Using Nix in Shebang
Traditionally, the shebang (`#!`) in shell scripts specifies the interpreter, commonly `/bin/sh` or `/bin/bash`. However, you can use any interpreter, like Python or PowerShell. With `nix-shell` as the interpreter, you can specify dependencies right before script execution.

Here's a Kubernetes Pod YAML to demonstrate:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nixos/nix
    command: ["/root/.nix-profile/bin/bash", "-c"]
    args:
    - |
      cp /etc/config/my-script.sh /run.sh && chmod +x $_
      /run.sh
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-script-configmap
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-script-configmap
data:
  my-script.sh: |
    #! /usr/bin/env nix-shell
    #! nix-shell -i bash -p kubectl
    kubectl --help
    tail -f /dev/null
```

This example is minimal but illustrates the core concept. More complex scripts can be written in the ConfigMap, and additional packages can be specified.

## Conclusion
By leveraging Nix and Kubernetes, you can make your container orchestration more efficient and reproducible. Whether you're building images or specifying dependencies in scripts, these methods offer a more declarative approach.
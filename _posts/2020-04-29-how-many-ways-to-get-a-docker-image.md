---
layout: post
title:  "How many ways to get a docker image?"
date:   2020-04-29 20:00:07 +0800
categories: docker image distribution
---

## 0. `docker pull`

It's the simplest way to get a image. The image come from a docker registry. The registry could be `hub.docker.com`, if you don't specify one.
You can learn how to pull a image by reading [the document](https://docs.docker.com/engine/reference/commandline/pull/).

- `docker pull nginx` will pull a image named `nginx` with the `latest` tag from `hub.docker.com`.
- `docker pull 192.168.2.3:5000/my/nginx:v0.2` will pull a image named `my/nginx` with the tag `v0.2` from `192.168.2.3:5000`.

## 1. `docker commit`

When you perform some manual operations in a running docker container, `docker commit` could pack the running container into a image. Read [the document](https://docs.docker.com/engine/reference/commandline/commit/)

- `docker commit my-service new-service-image` will make the running container named `my-service` into a image `new-service-image`.
- `docker commit ci23n50xjl new-service-image`, you can use the id of the container replace the name of container.

## 2. `docker build`

By writing the `Dockerfile`, you specify the infrastructure to make image. `docker build`

The `Dockerfile` may looks like this. You can build from a exist image or `scratch`.

```Dockerfile
FROM nginx
COPY ./web.conf /etc/web.conf
CMD nginx -c /etc/web.conf
```

- `docker build -t my-nginx .`. the current folder will be passed to docker, and the `Dockerfile` in current folder will be used to build image. The name of image will be `my-nginx`.
- `docker build -t my-nginx -f ./Dockerfile.arm64 .` you can specify a custom `Dockerfile`.

## 3. `docker load`

If you run `docker images`, you can see all images lay on your system.

You can save images into a file by using `docker save`

- `docker save image1 image2 -o images.tar`
- `docker save image3 | gzip > images.tar.gz`

You can get the images from the file at another host or time.

- `docker load -i images.tar.gz`

## 4. `docker import`

`docker import` will import a image from `rootfs`.

If you use a single binary without any libray, you make a `rootfs` with a single file.

```shell
mkdir rootfs
cp ./godemo ./rootfs
```

You'd better test the `rootfs` before import it to docker.

```shell
chroot ./rootfs /godemo
```

Now, import the `rootfs` to docker as a image named `my-image`.

`tar -C ./rootfs -c . | docker import - my-image`

If you want a distribution in `rootfs`, some tools can help.

### 4.1 `yum` for `CentOS`

We all know `yum` could install package to current host. It also could install packages into a single `rootfs`, like this.

`yum -y --installroot=<rootfs> install yum`

So with the srcrip below, you can make a simple centos image. You need specify `repo name`.

```shell
#!/bin/bash
set -eux

export ctr_root='/tmp/images/rootfs'
mkdir -p $ctr_root
yum -y --nogpgcheck --disablerepo=\* --enablerepo=<repo name> --releasever=7 --setopt=tsflags='nodocs' --setopt=override_install_langs=en_US.utf8 --installroot=$ctr_root install yum
tar -C $ctr_root -c . | docker import - my-centos-image
rm -rf $ctr_root
```

This way only works on Red Hat/CentOS. So we need another tool on Debian.

### 4.2 `debootstrap` for `Debian`

`debootstrap` is similar with the way we use `yum` above.

- `debootstrap xenial ./rootfs` , pack `xenial` version debian into folder `rootfs`.
- `debootstrap --arch=arm64 --foreign xenial rootfs http://ftp.cn.debian.org/debian/` , specify `arch` and a package repo.

if you mount `iso` to a folder, like this `mount -o loop debian.iso /tmp/debian`. you can get package from it.

- `debootstrap xenial rootfs file:///tmp/debian`

But `yum` and `debootstrap` only could make same serial distribution. How to make `ubuntu` image on `centos` system?

### 4.3 `squashfs` for any

First, install `squashfs-tools`. And download ubuntu iso image, `ubuntu-18.04.3-desktop-amd64.iso`.

Prepaire folder, `mkdir ubuntu rootfs`.

Mount iso, `mount -o loop ubuntu-18.04.3-desktop-amd64.iso ubuntu`

Make the `rootfs`, `unsquashfs -f -d ./rootfs/ ubuntu/casper/filesystem.squashfs`

Now you can make any distribution on any system.

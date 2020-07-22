---
layout: post
title: How to make a multi-architecture docker image
date: 2020-07-22 17:30
category: docker
author: m0sscode
tags: [docker, image, arch]
summary:
---

In fact, it's not an image. It's a list of images with arch or os labels.
With the help of the list, Docker could pull the right image which
matches different hosts by using the same "list name"/"image name".
Let's get the details.

Firstly, this function is experimental. So we need to enable it in the Docker config file.
A simple way is adding `"experimental": "enabled"` in `~/.docker/config.json`.

The "list" is called the "manifest list" in official documents. When we create it, there will
need a name of the list. The name will be used as a regular image name when we finish later.
And the content of the list is also needed when creating. Besides, the images in the list must __have
been pushed to registry__. The command create will check them. After pushing images, we could
create a manifest list which includes these images. The command of creating operation looks like this.

- `docker manifest create m0ssc0de/wg-go-ui:v1.0.2 m0ssc0de/wg-go-ui-amd64:v1.0.2 m0ssc0de/wg-go-ui-arm64:v1.0.2`

`m0ssc0de/wg-go-ui:v1.0.2` the first argument after `create` is the name of list.

All behind the list name is the images which will be included in the list.
- `m0ssc0de/wg-go-ui-amd64:v1.0.2`
- `m0ssc0de/wg-go-ui-arm64:v1.0.2`

Then we need to annotate the images with different labels. Just like this. Annotate one image once.

- `docker manifest annotate m0ssc0de/wg-go-ui:v1.0.2  m0ssc0de/wg-go-ui-amd64:v1.0.2 --arch amd64`
- `docker manifest annotate m0ssc0de/wg-go-ui:v1.0.2  m0ssc0de/wg-go-ui-arm64:v1.0.2 --arch arm64`

Then we can check the manifest list or not.

`docker manifest inspect m0ssc0de/wg-go-ui:v1.0.2`

Then, we can push the list.

`docker manifest push m0ssc0de/wg-go-ui:v1.0.2`

If there is any error, we could see the multi-arch "image" on registry.
And there is a tip from me. Add `--purge` to `push` command.
This will benefit updating manifest list. By adding `--purge`, the manifest list will be deleted after
pushing. Next time when we need to update this manifest list, we could just create a new one with new list
content. If not include `--purge`, the create action will combine the old list and new list. The part of old
list is useless for me. So we could make a simple pipeline to create a manifest list like this.

```shell
# 1. build all arch images and push
# docker push m0ssc0de/wg-go-ui-amd64:v1.0.2
# docker push m0ssc0de/wg-go-ui-arm64:v1.0.2

# 2. create manifest list
docker manifest creat m0ssc0de/wg-go-ui:v1.0.2 \
		m0ssc0de/wg-go-ui-amd64:v1.0.2 \
		m0ssc0de/wg-go-ui-arm64:v1.0.2

# 3. annotate image
docker manifest annotate m0ssc0de/wg-go-ui-amd64:v1.0.2 --arch amd64
docker manifest annotate m0ssc0de/wg-go-ui-arm64:v1.0.2 --arch arm64

# 4. push manifest
docker manifest push m0ssc0de/wg-go-ui:1.0.2 --purge

# 5. update latest
docker manifest creat m0ssc0de/wg-go-ui:latest\
		m0ssc0de/wg-go-ui-amd64:v1.0.2 \
		m0ssc0de/wg-go-ui-arm64:v1.0.2
docker manifest push m0ssc0de/wg-go-ui:latest --purge
```

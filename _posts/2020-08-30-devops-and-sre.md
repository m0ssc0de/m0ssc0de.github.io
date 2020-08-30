---
layout: post
title: DevOps and SRE (part I)
date: 2020-08-20 11:21
category: culture
author: m0sscode
tags: [devops, sre, git]
summary:
---

I will try my best to describe what is DevOps and SRE. And then point the difference between them.

## what is DevOps

Let's read the Wikipedia.

>  "a set of practices intended to reduce the time between committing a change to a system and the change being placed into normal production, while ensuring high quality"

The DevOps toolchain offered by the paper.

1. Coding – code development and review, source code management tools, code merging.
2. Building – continuous integration tools, build status.
3. Testing – continuous testing tools that provide quick and timely feedback on business risks.
4. Packaging – artifact repository, application pre-deployment staging.
5. Releasing – change management, release approvals, release automation.
6. Configuring – infrastructure configuration and management, infrastructure as code tools.
7. Monitoring – applications performance monitoring, end-user experience.

In my opinion, this is related to the two models, development model, and running model. They communicate with the artifacts. The artifacts should be tested well.
The final goal is to make business logical working and changing smoothly.

Most work is to define and implate these two models. Based on experience, it's better to start from the artifacts.

### Artifacts

What the artifacts should be is like the protocol of communication between the development model and the running model.

It depends on what the development model could output and what the running model needs. For example, if the development team doesn’t have any knowledge of docker, the artifacts could be a bunch of binary/jar/script, SQL, and config files. There should be a bunch of the document to describe what environment they need, how to change the config section which relates the environment, and how to start the application. These are the minimum requirements. Then add a document to describe how to upgrade the application with the artifacts. If the application is a platform, such as IaaS or PaaS, the artifacts maybe include some content files, such as images.

The next step is to make the artifacts more simple and convenient. This will depend on the improvement of the development model and running model.

So let's discuss the application and the two models.

### Running model

This model includes managed units and business logic. The managed unit is a minimal unit to manage. The operation of management could be started, stop, update, and so on. The business logic which the client need will come from the unit. The unit could be a systemd service, a pm2 script, a supervisor program, a docker container, or a k8s replica, CRD even Operator.

Base on different running environments, the artifacts will could be different types. They could be the traditional way, a group binary and config file looks like the one we mentioned above. It could be a group of docker images and a declarative definition. Yaml files, Helm Chart, or kustomize YAML files are better choices for k8s stack. Or other declarative definitions for different providers or tools, CloudFormation for AWS, Google Cloud Deployment Manager for GCP, HCL for terraform.

The new declarative way is better than the old one. Because it could include more information in it and reduce the toil. It could include what the unit is and how to manage it, not only a mess of black box for others. It also could include the relation of the units. It will be helpful to implement a cloud-native service mesh.

And then we should know the status of the running model. So there is a monitoring system. In my opinion, there will be 3 layer monitoring, the host layer, the app layer, and the business layer. The host layer could be the monitoring of a group Linux host. Or it could be the status of a managed platform, such as a k8s, an AWS container service, or a GCP FaaS platform. The app layer is the status of a managed unit. It refers to the status of the process, which may include start time, usage of CPU, memory, and storage. The business layer will monitor business logic. It records the number and the quality of the business logic works. It will be good for knowing the status of the running model and improving the business logic self.

### Development model

This model starts with predefined business logic. The logic may not be perfect and immutable at the beginning. But it will be great for being written as clear as possible. Then create a design to run the business logic. In this process, a lot of communication and trade-offs are required. This could be included in another blog. We only talk about the flow process today.

After the design, we'd better make a plan to implement it. It will include how we write code and how the git-flow looks like. We need to define some processes based on the technical capabilities of the team. It’s necessary to sync upstream branch by merge or rebase, no mater local branch or remote branch. Then some automation processes and a review are also required before merging to the upstream.

Most automation solutions base on webhooks in the git server. This will give us opportunities to add automation action based on the trigger. It gives us opportunities to check, test, integrate, deliver. Finally, the project could release high-quality Artifacts.

In short, DevOps considers both the development phase and the operation phase in order to work faster and smoother. It can fill some gaps and think from different perspectives. Of course, this also brings some challenges. It needs to consider the needs and actual conditions of different fields, and communicate with different teams. There are many interesting things to play with.

The next part will introduce my view of SRE. Waiting for your review.


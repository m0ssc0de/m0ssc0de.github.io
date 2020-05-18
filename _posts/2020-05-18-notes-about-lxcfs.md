---
layout: post
title: Notes about lxcfs
date: 2020-05-18 10:01
category:
author: m0ssc0de
tags: [lxcfs, docker, cgroup, k8s, kubernetes]
summary:
---

Get a task to add `lxcfs` on `mips64le` server. So it's a good chance to keep a summary about `lxcfs`.

Firstly, we try it on amd64 server.

Setup the lab by running `curl -fsSL https://get.docker.com | sh -`

Get the total number of memory by `/proc`.

```shell
$ cat /proc/meminfo | grep MemTotal
MemTotal:       49293096 kB
```

Just run a container, and cat memory info.

```shell
$ docker run -it --rm centos bash
[root@ae6aa1605e85 /]$ cat /proc/meminfo | grep MemTotal
MemTotal:       49293096 kB
```

They are same. How about run container with memory limit.

```shell
$ docker run -it --rm -m 80000000 centos bash
[root@acf3b4d00136 /]$ cat /proc/meminfo | grep MemTotal
MemTotal:       49293096 kB
```

They are same too.

Keep container running. Let's make sure setting take effect.

```shell
$ docker inspect acf3b | grep Memory
            "Memory": 80000000,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 160000000,
            "MemorySwappiness": null,
```

So docker have received our `80000000`. But what effect does this different? Let's run a test.

Create a container by running `docker run -it --rm -m 10485760 --privileged centos bash`

- `-m 10485760` means that assigns 10M memory to this container. If the process in container use memory over this limit, it will be killed.
- `--privileged` means that container will have more privileged power. Then we will could have chance to simulate memory usage.


```shell
$ docker run -it --rm -m 10485760 --privileged centos bash
[root@f91146b168a2 /]$ mkdir testram && mount -t ramfs ramfs testram/
[root@f91146b168a2 /]$ dd if=/dev/zero of=z/file bs=1M count=5
5+0 records in
5+0 records out
5242880 bytes (5.2 MB, 5.0 MiB) copied, 0.00326266 s, 1.6 GB/s
```
Now we could see memory were used by about 5M. Let try more.

```shell
[root@f91146b168a2 /]$ dd if=/dev/zero of=testram/file bs=1M count=100
Killed
```

So if a process read memory information from `/proc/meminfo`, it will get a different number from docker setting for this process. The process will be killed, when process use more memory than the limit from docker.

Unfortunately the jvm from java runs like this way. Though jvm could access memory limit from launch command, container should have more isolation to make user more convenience.

And lxcfs is to solve this problem by reading [doc](https://linuxcontainers.org/lxcfs/introduction/)

It separate container's `/proc` from host's one by useing a virtual filesystem. And this make the process in container have chance to read the `real` memory info.

Let's compile and install lxcfs on host

```shell
$ yum install automake libtool fuse-devel pam-devel
$ git clone git://github.com/lxc/lxcfs && cd lxcfs
$ git checkout lxcfs-2.0.8
$ ./bootstrap.sh && ./configure
$ mkdir -p /var/lib/lxcfs && make && make install
$ cat << EOF | tee /usr/lib/systemd/system/lxcfs.service
[Unit]
Description=FUSE filesystem for LXC
ConditionVirtualization=!container
Before=lxc.service
Documentation=man:lxcfs(1)

[Service]
ExecStart=/usr/local/bin/lxcfs /var/lib/lxcfs/
KillMode=process
ExecStopPost=-/bin/fusermount -u /var/lib/lxcfs
Delegate=yes

[Install]
WantedBy=multi-user.target
EOF
$ systemctl enable lxcfs && systemctl start lxcfs
```

And now run a new container with lxcfs.

```shell
$ docker run -it --rm --privileged=true \
    -m 10485760 \
    -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo \
    centos bash
[root@72e7e514827e /]$ cat /proc/meminfo | grep Mem
MemTotal:          10240 kB
MemFree:            7544 kB
MemAvailable:       7544 kB
```

We can see 10M memory limit had effect the `/proc`. You can make more isolation by add more mount parameters like this.

```shell
$ docker run -it --rm --privileged=true \
    --cpuset-cpus=2-9 -m 1024m \
    -v /var/lib/lxcfs/proc/uptime:/proc/uptime:rw \
    -v /var/lib/lxcfs/proc/cpuinfo:/proc/cpuinfo:rw \
    -v /var/lib/lxcfs/proc/stat:/proc/stat \
    -v /var/lib/lxcfs/cgroup/:/cgroup/:rw \
    -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo \
    -v /var/lib/lxcfs/proc/swaps:/proc/swaps \
    -v /var/lib/lxcfs/proc/diskstats:/proc/diskstats \
    centos bash
```

So far, we have learn how to use docker with lxcfs. Next section, we will combine lxcfs and kubernetes.
---
layout: post
title:  "A transparent proxy on Raspberry"
date:   2020-05-4 20:00:07 +0800
categories: proxy raspberry
---

I have wast so many time on GFW. This is an another note about setup a proxy. There will not be more details and disscustion on this topic. There will be just some script and config.

## System

1. `touch ssh` on root at `system-root` of Raspberry Ubuntu.

2.  Config static ip by edit `/etc/netplan/50-cloud-init.yaml`. `netplan apply` to apply.

```config
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: false
            addresses: [192.168.1.202/24]
            gateway4: 192.168.1.1
            nameservers:
              addresses: [192.168.1.1]
    version: 2
```

3. Change system DNS(`/etc/resolv.conf`) to `192.168.1.202`

## Clash

1. Copy right arch clash binary to `/usr/bin/clash`.

2. Place clash config file at `/root/.config/clash/config.yaml`

maybe need this config

```config
external-controller: 192.168.2.2:9090
dns:
  enable: true
  ipv6: false
  listen: 192.168.2.2:53
  #enhanced-mode: fake-ip
  enhanced-mode: redir-host
  nameserver:
    - 114.114.114.114
    - 223.5.5.5
    - tls://dns.rubyfish.cn:853
  fallback:
    - 114.114.114.114
    - tls://dns.rubyfish.cn:853
    - 8.8.8.8
```

3. Place systemd config file for clash. `/usr/lib/systemd/system/clash@.service`

```config
[Unit]
Description=A rule based proxy in Go for %i.
After=network.target

[Service]
Type=simple
User=%i
Restart=on-abort
ExecStart=/usr/bin/clash

[Install]
WantedBy=multi-user.target
```

```shell
systemctl start clash@root
systemctl enable clash@root
systemctl stop clash@root
```

## Router

1. Manual test

Config your phone or something's `gateway` and `dns` to the Raspberry. Test internet.

2. If everything ok. Change router's DHCP. Set Raspberry's address as default `gateway` and `dns`.

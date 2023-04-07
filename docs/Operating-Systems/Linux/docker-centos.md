---
title: Install Docker
description: on CentOS 8
published: true
date: 2020-05-15T21:55:46.563Z
tags:
---

# Install Docker
Follow https://docs.docker.com/engine/install/centos/#install-docker-engine

# Install Docker Compose
Follow https://docs.docker.com/compose/install/#install-compose

# Setup Firewall
From https://serverfault.com/a/994704

1) Check what interface docker is using, e.g. 'docker0'
```shell
ip link show
```

2) Check available firewalld zones, e.g. 'public'
```shell
sudo firewall-cmd --get-active-zones
```

3) Check what zone the docker interface it bound to, most likely 'no zone' yet
```shell
sudo firewall-cmd --get-zone-of-interface=docker0
```

4) So add the 'docker0' interface to the 'public' zone. Changes will be visible only after firewalld reload
```shell
sudo nmcli connection modify docker0 connection.zone public
```

5) Masquerading allows for docker ingress and egress (this is the juicy bit)
```shell
sudo firewall-cmd --zone=public --add-masquerade --permanent
```

6) Reload firewalld and dockerd
```shell
sudo firewall-cmd --reload
sudo systemctl restart docker
```

7) Test ping and DNS works:
```shell
docker run busybox ping -c 1 8.8.8.8
docker run busybox ping -c 1 vinrobot.net
docker run busybox cat /etc/resolv.conf
```

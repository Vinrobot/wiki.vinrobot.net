---
title: NVMe/TCP (WIP)
description:
published: false
date: 2025-06-09T20:00:00.000Z
tags:
---

https://spdk.io/doc/bdev.html
https://spdk.io/doc/nvmf.html
https://github.com/vifino/nvmetcfg

# Install SPDK

```shell
$ git clone https://github.com/spdk/spdk --recursive
$ cd spdk/
$ sudo ./scripts/pkgdep.sh --all
$ sudo apt-get install python3 ninja-build meson python3-pyelftools
$ sudo pip3 install --break-system-packages pyelftool
$ ./configure --with-rdma --with-raid5f
$ make -j$(nproc)
```

```shell
$ sudo ./scripts/setup.sh reset
$ sudo HUGEMEM=4096 scripts/setup.sh
$ sudo ./scripts/setup.sh status
```

# Configure SPDK

```shell
./build/bin/nvmf_tgt &
```

## Attach NVMe controllers

```shell
./scripts/setup.sh status
./scripts/rpc.py bdev_nvme_attach_controller -b Nvme0 -t PCIe -a 0000:02:04.0
```

## Setup RAID

```shell
./scripts/rpc.py bdev_raid_create -n Raid5 -r 5f -b "Nvme0n1 Nvme0n2 Nvme0n3 Nvme0n4" -z 1
```

## Setup NVMe/TCP target

```shell
./scripts/rpc.py nvmf_create_transport -t TCP -u 16384 -m 8 -c 8192
./scripts/rpc.py nvmf_create_subsystem nqn.2016-06.io.spdk:cnode1 -a -s SPDK00000000000001 -d "SPDK NVMe over Fabrics Target"
./scripts/rpc.py nvmf_subsystem_add_ns nqn.2016-06.io.spdk:cnode1 Raid5
./scripts/rpc.py nvmf_subsystem_add_listener nqn.2016-06.io.spdk:cnode1 -t TCP -a 192.168.202.200 -s 4420
```

## Backup and Restore

```shell
$ ./scripts/rpc.py save_config > config.json
```

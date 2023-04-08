---
title: Setup LVM for Ubuntu
---

# Setup LVM for Ubuntu

Ubuntu Desktop installer doesn't allow to create a LVM installation manually.
Therefore you have to setup LVM before starting the installer (i.ex: using the Try Ubuntu option).

We are assuming the disk is `/dev/nvme0n1`.

## Setup disk partitions

1) Zero the partition table
```shell
$ sudo sgdisk -Z /dev/nvme0n1
```

2) Create partitions

```shell
$ sudo sgdisk -n 1:0:+512M -t 1:ef00 -c "1:EFI System" /dev/nvme0n1
$ ls /dev/nvme0n1p1
$ sudo sgdisk -n 2:0:0 -t 2:8e00 -c "2:Linux LVM" /dev/nvme0n1
$ ls /dev/nvme0n1p2
```

Common partition types from [archlinux wiki: GPT fdisk](https://wiki.archlinux.org/title/GPT_fdisk#Partition_type){target=_blank}:

* `ef00`: EFI System
* `8e00`: Linux LVM

3) Format EFI partition

```shell
$ sudo mkfs.fat -F 32 /dev/nvme0n1p1
```

## Setup LVM

1) Create a physical volume

```shell
$ sudo pvcreate /dev/nvme0n1p2
```

2) Create a volume group

```shell
$ sudo vgcreate vg0 /dev/nvme0n1p2
```

3) Create logical volumes

```shell
$ sudo lvcreate -Z y -L 1GB --name boot vg0
$ sudo lvcreate -Z y -l 100%FREE --name root vg0
```

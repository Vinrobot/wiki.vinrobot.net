---
title: Install a complete PXE server
description: on CentOS
published: true
date: 2020-03-18T21:42:44.572Z
tags:
---

# Configure DHCP

Source: [CentOS 8 : PXE Boot](https://www.server-world.info/en/note?os=CentOS_8&p=pxe)
Subnet: 192.168.20.0/24

## Install DHCP-Server
```
# dnf -y install dhcp-server
```

## Setup DHCP-Server (1/2)
Setup a simple DHCP server and tests it.
```
# cat /etc/dhcp/dhcpd.conf
option domain-name "vr.local";
option domain-name-servers 192.168.20.2;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.20.0 netmask 255.255.255.0 {
	range dynamic-bootp 192.168.20.200 192.168.20.254;
	option broadcast-address 192.168.20.255;
	option routers 192.168.20.2;
}
```

```
# systemctl enable --now dhcpd
```

## Setup DHCP-Server (2/2)
Add the PXE configuration to the DHCP.
```
# cat /etc/dhcp/dhcpd.conf
...
option space pxelinux;
option pxelinux.magic code 208 = string;
option pxelinux.configfile code 209 = text;
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option architecture-type code 93 = unsigned integer 16;

subnet 192.168.20.0 netmask 255.255.255.0 {
	...

	class "pxeclients" {
		match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
		next-server 192.168.20.50;

		if option architecture-type = 00:07 {
			filename "BOOTX64.EFI";
		} else {
			filename "syslinux/pxelinux.0";
		}
	}
}
```

```
# systemctl restart dhcpd
```

# Configure TFTP

## Install TFTP-Server
```
# dnf -y install tftp-server
# systemctl enable --now tftp.socket
# firewall-cmd --add-service=tftp --permanent
# firewall-cmd --reload
```

## Configure SYSLINUX
Configure the SYSLINUX bootloader.
```
# dnf -y install syslinux
# mkdir /var/lib/tftpboot/syslinux
# cp /usr/share/syslinux/{pxelinux.0,menu.c32,vesamenu.c32,ldlinux.c32,libcom32.c32,libutil.c32} /var/lib/tftpboot/syslinux/
# mkdir /var/lib/tftpboot/syslinux/pxelinux.cfg
```

```
# cat /var/lib/tftpboot/syslinux/pxelinux.cfg/default
default vesamenu.c32
prompt 1
timeout 60

display boot.msg

label local
  menu label Boot from ^local drive
  localboot 0xffff
```

Older versions:
- [SYSLINUX official releases](https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/)
- [SYSLINUX RPM packages](https://rpmfind.net/linux/rpm2html/search.php?query=syslinux&system=dag&arch=x86_64)

## Configure GRUB2 (WIP)

- https://docs.oracle.com/cd/E64076_01/E64078/html/vmiug-appendix-pxe-boot.html#vmiug-install-pxe-uefi-setup
- https://xenappblog.com/2018/automatically-install-vmware-esxi-6-7/
- https://www.server-world.info/en/note?os=CentOS_8&p=pxe&f=3
- http://www.manobit.com/pxe-multi-boot-server-using-grub2-on-mikrotik-routeros-bios-and-efi-support/#BIOSUEFI
- https://docs.oracle.com/cd/E64076_01/E64078/html/vmiug-appendix-pxe-boot.html
- https://communities.vmware.com/thread/520508

## Configure iPXE (WIP)

- https://ipxe.org/howto/chainloading
- https://ipxe.org/cfg/platform
- https://gist.github.com/nemf/3794614
- https://gist.github.com/robinsmidsrod/08105908079357b2dab5
- https://forum.ipxe.org/showthread.php?tid=1123

# Add systems

## CentOS 8
Mount CentOS 8 ISO:
```
# cat /etc/fstab
...
/var/iso/CentOS-8.1.1911-x86_64-dvd1.iso /mnt/centos8 iso9660 loop,ro 0 0

# mkdir /var/lib/tftpboot/centos8 /mnt/centos8
# mount -a
# cp /mnt/centos8/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/centos8/
```

Add menus to SYSLINUX:
```
# cat /var/lib/tftpboot/syslinux/pxelinux.cfg/default
...

label linux
  menu label ^Install CentOS 8
  menu default
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img ip=dhcp inst.repo=http://192.168.20.50/centos8
label vesa
  menu label Install CentOS 8 with ^basic video driver
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img ip=dhcp inst.xdriver=vesa nomodeset inst.repo=http://192.168.20.50/centos8
label rescue
  menu label ^Rescue installed system
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img rescue

label local
  ...
```

Provide image files through Apache:
```
# cat /etc/httpd/conf.d/pxeboot.conf
...
Alias /centos8 /mnt/centos8
<Directory /mnt/centos8>
    Options Indexes FollowSymLinks
    Require ip 127.0.0.1 192.168.20.0/24
</Directory>
...
```

## VMware ESXi 6.0
Source: [Installing ESXi Using PXE](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/vsphere-esxi-vcenter-server-60-pxe-boot-esxi.pdf)
Tested SYSLINUX versions: 3.86, 4.07 (with gpxelinux.0)

---

Mount VMware ESXi 6.0 ISO:
```
# cat /etc/fstab
...
/var/iso/VMware-ESXi-6.0.0.iso /mnt/vmware-esxi-6 iso9660 loop,ro 0 0

# mkdir /var/lib/tftpboot/vmware-esxi-6 /mnt/vmware-esxi-6
# mount -a
# cp /mnt/vmware-esxi-6/{mboot.c32,boot.cfg} /var/lib/tftpboot/vmware-esxi-6/
# vim /var/lib/tftpboot/vmware-esxi-6u3-hpe/boot.cfg
Set prefix=http://192.168.20.50/vmware-esxi-6/
And remove "/" from kernel and modules
```

Add menus to SYSLINUX:
```
# cat /var/lib/tftpboot/syslinux/pxelinux.cfg/default
...

label vwareesxi6
  menu label Install VMware ESXi 6.0
  kernel vmware-esxi-6/mboot.c32
  append -c vmware-esxi-6/boot.cfg
  ipappend 2

label local
  ...
```

Provide image files through Apache:
```
# cat /etc/httpd/conf.d/pxeboot.conf
...
Alias /vmware-esxi-6 /mnt/vmware-esxi-6
<Directory /mnt/vmware-esxi-6>
    Options Indexes FollowSymLinks
    Require ip 127.0.0.1 192.168.20.0/24
</Directory>
...
```

---
title: Set FEC mode
description:
published: true
date: 2026-10-11T22:00:00.000Z
tags:
---

**This guide expects you to have a working RPM-based distro with the NVIDIA Firmware Tools installed.**

See [Install NVIDIA Firmware Tools](install-mft.md)

---

1. Identify your MST device path

```shell
$ sudo mst status
MST modules:
------------
    MST PCI module is not loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4117_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:02:01.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 00
```

The device path is `/dev/mst/mt4117_pciconf0`.

2. Check the current values

```shell
$ sudo mlxconfig -d /dev/mst/mt4117_pciconf0 query | grep PHY_FEC_OVERRIDE
        PHY_FEC_OVERRIDE_P1                             DEVICE_DEFAULT(0)
        PHY_FEC_OVERRIDE_P2                             DEVICE_DEFAULT(0)
```

3. Check the valid values and their meanings

```shell
$ sudo mlxconfig -d /dev/mst/mt4117_pciconf0 i | grep -A10 PHY_FEC_OVERRIDE
                    PHY_FEC_OVERRIDE_P1=<DEVICE_DEFAULT|MODE_1|MODE_2|MODE_3|MODE_4>Customizes FEC configuration of the port
                                                            0x0: DEVICE_DEFAULT
                                                            0x1: MODE_1 - disabled DME, disabled FEC for 25G/50G/100G
                                                            0x2: MODE_2 - disabled DME, RS FEC for 25G/50G/100G
                                                            0x3: MODE_3 - disabled DME, FC FEC for 25G/50G or RS FEC for 100G
                                                            0x4: MODE_4 - disable non_dme (enable Auto Negotiation)
                                                            Other values are reserved
```

`RS FEC` = `CL91` = `FE91` = ... (don't quote me on that)

4. Set the mode

```shell
$ sudo mlxconfig -d /dev/mst/mt4117_pciconf0 set PHY_FEC_OVERRIDE_P1=2
```

5. Reboot

6. Verify

```shell
$ sudo mst start
$ sudo mlxconfig -d /dev/mst/mt4117_pciconf0 q PHY_FEC_OVERRIDE_P1
```

---
title: Retrieve Windows OEM Product Key
description:
published: true
date: 2020-02-07T22:55:06.830Z
tags:
---

# from Linux
```
# strings /sys/firmware/acpi/tables/MSDM | tail -1
```

# from Windows
```batch
wmic path softwarelicensingservice get OA3xOriginalProductKey
```

# Links
- https://www.cyberciti.biz/faq/linux-find-windows-10-oem-product-key-command/

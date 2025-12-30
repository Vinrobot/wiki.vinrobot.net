---
title: Debug failing hardware
description:
published: true
date: 2025-12-30T17:30:00.000Z
tags:
---

# Serial console

- Configuration: `115200/8/N/1`

- [wrgms.com - Recovering a Failed Synology Diskstation with a Serial Console](https://wrgms.com/recovering-a-failed-synology-diskstation-ds2xx-serial/)

# Telnet password

- [wrgms.com - Synology's "secret" telnet password](https://wrgms.com/synologys-secret-telnet-password/)

- Password generator code:
```c
#include <stdlib.h>
#include <time.h>
#include <stdio.h>

int gcd(int a, int b)
{
    return (b ? gcd(b, a % b) : a);
}

void main()
{
    struct timeval now;
    struct tm time;

    gettimeofday(&now, 0);
    localtime_r(&(now.tv_sec), &time);

    time.tm_mon += 1;
    printf("Password: %x%02d-%02x%02d\n", time.tm_mon, time.tm_mon, time.tm_mday, gcd(time.tm_mon, time.tm_mday));
}
```

- Password for Jan 01: `101-0101`

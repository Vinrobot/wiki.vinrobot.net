---
title: Launchd
description:
published: true
date: 2020-02-07T23:13:32.723Z
tags:
---


# Job Definitions

Job definitions crucial for the operation of the operating system are stored below /System/Library. You should never need to create a daemon or agent in these directories.

Third-Party definitions which are relevant for every user are stored below /Library. Job definitions for a specific user are stored below the respective user's Library directory.

| Type           | Location                      | Run on behalf of                                 |
|----------------|-------------------------------|--------------------------------------------------|
| User Agents    | ~/Library/LaunchAgents        | Currently logged in user                         |
| Global Agents  | /Library/LaunchAgents         | Currently logged in user                         |
| Global Daemons | /Library/LaunchDaemons        | root or the user specified with the key UserName |
| System Agents  | /System/Library/LaunchAgents  | Currently logged in user                         |
| System Daemons | /System/Library/LaunchDaemons | root or the user specified with the key UserName |

# Links
- https://www.launchd.info

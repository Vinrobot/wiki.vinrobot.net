---
title: Remove segments
description:
published: true
date: 2020-02-09T13:25:21.468Z
tags:
---

## Locate segments
Locate the start and end of the segments you want to delete
```bash
$ ffmpeg -y -ss 00:25:53.30 -i abc.mkv -frames:v 1 out.jpg
$ ffmpeg -y -ss 00:25:59.28 -i abc.mkv -frames:v 1 out.jpg
```
Segment to delete: `00:25:53.30` to `00:25:59.28`

## Create segments
Create segments `a` and `c` (the video without the segments you want to delete)
```bash
$ ffmpeg -i abc.mkv -map 0 -c copy -t 00:25:53.30 a.mkv
$ ffmpeg -ss 00:25:59.28 -i abc.mkv -map 0 -c copy c.mkv
```

## Join segments
Create a file listing the segments created previously
```bash
$ cat files.txt
file 'a.mkv'
file 'c.mkv'
```

Concatenate the segments
```bash
$ ffmpeg -f concat -i files.txt -map 0 -c copy ac.mkv
```

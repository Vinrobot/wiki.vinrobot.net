---
title: Homebre Bundle
description:
published: true
date: 2020-02-07T23:23:25.799Z
tags:
---

# Homebrew
## Create from Installed packages
```bash
$ brew leaves > brews.txt
```

## Install
```bash
$ xargs brew install < brews.txt
```

# Homebrew Bundle
## Create from Installed packages
```bash
$ brew bundle dump
```

## Install
```bash
$ brew bundle
or
$ brew bundle --file ${FILE}
```

# Links
- https://github.com/Homebrew/homebrew-bundle
- https://apple.stackexchange.com/q/279077

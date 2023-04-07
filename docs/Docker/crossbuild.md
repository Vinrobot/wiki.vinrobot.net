---
title: Build multi-arch Docker images
description:
published: true
date: 2020-05-17T18:13:15.840Z
tags:
---

Source:
- https://mirailabs.io/blog/multiarch-docker-with-buildx/
- https://docs.docker.com/buildx/working-with-buildx/

## Enable buildx
```shell
$ export DOCKER_CLI_EXPERIMENTAL=enabled
$ docker buildx version
github.com/docker/buildx v0.3.1-tp-docker 6db68d029599c6710a32aa7adcba8e5a344795a7
```

## Enable binfmt (Linux only)
```shell
$ docker run --rm --privileged docker/binfmt:a7996909642ee92942dcd6cff44b9b95f08dad64
$ ls -al /proc/sys/fs/binfmt_misc/
total 0
drwxr-xr-x. 2 root root 0 May 17 19:49 .
dr-xr-xr-x. 1 root root 0 May 17 19:38 ..
-rw-r--r--. 1 root root 0 May 17 19:49 qemu-aarch64
-rw-r--r--. 1 root root 0 May 17 19:49 qemu-arm
-rw-r--r--. 1 root root 0 May 17 19:49 qemu-ppc64le
-rw-r--r--. 1 root root 0 May 17 19:49 qemu-riscv64
-rw-r--r--. 1 root root 0 May 17 19:49 qemu-s390x
--w-------. 1 root root 0 May 17 19:49 register
-rw-r--r--. 1 root root 0 May 17 19:49 status
$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
enabled
interpreter /usr/bin/qemu-aarch64
flags: OCF
offset 0
magic 7f454c460201010000000000000000000200b7
mask ffffffffffffff00fffffffffffffffffeffff
```

## Create the multi-arch builder
```shell
$ docker buildx ls
NAME/NODE DRIVER/ENDPOINT STATUS  PLATFORMS
default * docker
  default default         running linux/amd64, linux/386
$ docker buildx create --use --name cross-builder
cross-builder
$ docker buildx ls
NAME/NODE        DRIVER/ENDPOINT             STATUS   PLATFORMS
cross-builder *  docker-container
  cross-builder0 unix:///var/run/docker.sock inactive
default          docker
  default        default                     running  linux/amd64, linux/38
```

## Tests
1) Create a Go file
```go
$ cat hello.go
package main

import (
        "fmt"
        "runtime"
)

func main() {
        fmt.Printf("Hello, %s!\n", runtime.GOARCH)
}
```

2) Create a Dockerfile
```dockerfile
FROM golang:alpine AS builder
RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN go build -o hello .

FROM alpine
RUN mkdir /app
WORKDIR /app
COPY --from=builder /app/hello .
CMD ["./hello"]
```

3) Build using buildx
```shell
$ docker buildx build -t hello-multi-arch --platform=linux/arm,linux/arm64,linux/amd64,windows/amd64 .
```


## Set the default builder to buildx
https://docs.docker.com/buildx/working-with-buildx/#set-buildx-as-the-default-builder

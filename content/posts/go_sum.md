---
title: "go1.16 docker build遇到的问题"
date: 2022-09-23T14:11:27+08:00
draft: false
categories: ["golang","docker"]
---

在我本机能运行的Dockerfile，在同事机器上build步骤会失败  
Dockerfile:
```
FROM golang:1.16 AS builder

ARG APP_RELATIVE_PATH

COPY . /src
WORKDIR /src/app/${APP_RELATIVE_PATH}

RUN GOPROXY=https://goproxy.cn make build

FROM debian:stable-slim

ARG APP_RELATIVE_PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
		ca-certificates  \
        netbase \
        && rm -rf /var/lib/apt/lists/ \
        && apt-get autoremove -y && apt-get autoclean -y

COPY --from=builder /src/app/${APP_RELATIVE_PATH}/bin /app

WORKDIR /app

EXPOSE 8000
EXPOSE 9000
VOLUME /data/conf

CMD ["./server", "-conf", "/data/conf"]
```
Makefile:
```
.PHONY: build
# build
build:
	mkdir -p bin/ && go build -ldflags "-X main.Version=$(VERSION)" -o ./bin/ ./...

```
帮他排查问题,报错信息:
```
     => ERROR [builder 4/4] RUN GOPROXY=https://goproxy.cn make build                                                                                                                                                            4.5s
------
 > [builder 4/4] RUN GOPROXY=https://goproxy.cn make build:
#11 0.554 mkdir -p bin/ && go build -ldflags "-X main.Version=42ad07a" -o ./bin/ ./...
#11 4.415 go: github.com/spf13/cast@v1.5.0 requires
#11 4.415       github.com/frankban/quicktest@v1.14.3: missing go.sum entry; to add it:
#11 4.415       go mod download github.com/frankban/quicktest
#11 4.422 make: *** [../../../app_makefile:72: build] Error 1
------
executor failed running [/bin/sh -c GOPROXY=https://goproxy.cn make   build]: exit code: 2
make: *** [docker] Error 1
```
google一下，用go mod tidy解决  
还是报错  
看下go env
goproxy，没有中国代理，加上之后  
还是报错  
陷入僵局了，将我本地的go mod，go sum copy 给他  
还是报错  
再googel一下，发现一个相关[issue](https://github.com/google/wire/issues/299)
```
    GOFLAGS=-mod=mod go generate ./...
```
再追踪进去[issue](https://github.com/golang/go/issues/44129)
```
    go build -mod=mod
```
根据这个指令改动一下build
```
    cd ../../.. && docker build -mod=mod -f deploy/build/Dockerfile  
 --build-arg APP_RELATIVE_PATH=$(APP_RELATIVE_PATH) -t $(DOCKER_IMAGE) .
```
成功了  

确实是go1.16版本bug导致的

思考： -mod=mod 为什么能成功？  
[stackoverflow](https://stackoverflow.com/questions/71121641/what-does-this-command-do-goflags-mod-mod)  
-mod=mod go尝试更新拉取gomod，看来只能定义成1.16的go mod tidy指令有bug  
[官方解释](https://go.dev/ref/mod#authenticating):
```
The go command starts by searching the build list for modules with paths that are prefixes  
of the package path. For example, if the package example.com/a/b is imported, and   
the module example.com/a is in the build list, the go command will check whether  
example.com/a contains the package, in the directory b. At least one file with the .go  
extension must be present in a directory for it to be considered a package. Build   
constraints are not applied for this purpose. If exactly one module in the build list  
provides the package, that module is used. If no modules provide the package or if two or  
more modules provide the package, the go command reports an error. The -mod=mod flag  
instructs the go command to attempt to find new modules providing missing packages and to  
update go.mod and go.sum. The go get and go mod tidy commands do this automatically.
```






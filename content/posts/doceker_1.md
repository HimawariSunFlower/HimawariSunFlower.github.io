---
title: "docker 初体验"
date: 2022-09-20T11:11:27+08:00
draft: true
categories: ["docker"]
---

1. 传进去的配置文件
    如果是监听端口，应当是0.0.0.0
    如果是外部服务，应当是对应ip，无法直接用0.0.0.0去连接宿主机  
2. Dockerfile 
3. docker 指令 
4. volumes 挂在相对路径的话，根节点在dockerfile的文件或者docker-compose.yml的路径
5. [🔗](https://blog.csdn.net/qq_38983728/article/details/98741935) cmd使用[]形式不能取到环境变量 [更具体的原理](https://yeasy.gitbook.io/docker_practice/image/dockerfile/cmd)
6. 多步构建
7. copy会跟在run后面执行
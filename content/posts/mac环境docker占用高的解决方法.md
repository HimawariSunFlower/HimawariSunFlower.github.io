---
title: "mac环境docker占用高的解决方法"
date: 2023-02-25T16:01:27+08:00
draft: false
categories: ["docker"]
---
# mac环境docker占用高的解决方法

docker在macbook里内存占用很高，监视器里面hyperkit动不动就是8，9个g的占用，估计是兼容还是做的不太好。想法是跑个虚拟机，再在虚拟机里跑docker。

## 虚拟机
虚拟机选用的ubuntu推出的的multipass，[下载地址](https://multipass.run/docs/installing-on-macos)。
安装之后 multipass launch 创建虚拟机，multipass shell vmname 进入虚拟机。

### 如何ssh连接虚拟机
进入vm之后发现默认已经装好了ssh-server。
ubuntu下默认是不允许root通过密码的方式通过ssh远程登录服务器的
```
sudo vi /etc/ssh/sshd_config
#增加以下配置允许通过ssh密码登录

#PermitRootLogin prohibit-password
PermitRootLogin yes

#PasswordAuthentication no
PasswordAuthentication yes

#修改完成后需要重启ssh服务命令如下
sudo service ssh restart
```

下面说下如何修改root密码

```
sudo passwd root
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully

su root #修改密码生效验证
#在下方输入修改后的密码，输入后回车
Password:

#成功
root@vm1:/home/ubuntu#

#失败
su: Authentication failure
```
至于ip的话可以用ifconfig在vm内部查看，也可以 multipass info vmname 在外部查看。
经过上面的配置，就可以ssh连接vm了。

```
ssh root@ip
root@ip's password:
```


## 运行docker

### 安装docker
这一步其实就和linux安装docker没有区别了,随便查查教程就行了，比如[这个](https://blog.csdn.net/qq_43648470/article/details/108171050)


### 管理docker
虚拟机里的docker没有了mac的可视化管理工具，很难受。这个时候Portainer.io英雄登场。轻量级的web可视化docker管理工具。[安装教程](https://zhuanlan.zhihu.com/p/256469146)

## 效果
再用监视器看内存，hyperkit1.68g,风扇也没有一直吵了，win。


# 后续产生问题

1. vm突然无法联网  
：开过🪜的话重启下网络

2. 直接访问vm里的consul去调用vm里的服务是没问题的，但是本机debug启动服务去注册进vm的consul，通过访问本机服务，本机服务再去consul调用其他服务会超时没有返回。  
jaeger里找不到被调用服务的链路记录，但是被调用服务有consul的日志  
```
INFO ts=2023-02-25T15:21:23+08:00 caller=discovery/resolver.go:84 service.id=service1@8fd3ad5f61a6 service.name=service1 service.version=ff59666 trace.id= span.id= msg=[resolver] update instances: [{"id":"service2@macdeMacBook-Pro.local","name":"service2","version":"","metadata":null,"endpoints":["grpc://service2"]}]
```
：还在排查

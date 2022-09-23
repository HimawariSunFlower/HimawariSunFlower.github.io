---
title: "go-kratos 实际开发问题"
date: 2022-09-19T11:11:27+08:00
draft: true
categories: ["go-kratos"]
---

1. repo层抽象优化问题
```
    https://lailin.xyz/post/graceful-repo-code.html
    关于repo层的接口优雅，抽象

    kratos的biz层，data层不太能处理？
        gorm这种具体orm不应当暴露给biz
        biz不能调用data会相互引用

    暂时没有思考出好的方式
```

2. ddd相关及kratos版ddd相关



3. 与dart客户端的grpc对接
   1. 返回error 他那边是try catch到再用grpcError类解析处理，等同于go的grpc包里的toRpcErr()
   2. rpc分类：  
    ![这是图片](/img/3.png)   
   3. 
   4. 
---
title: "vue生产环境跨域不生效"
date: 2023-04-07T14:51:27+08:00
draft: false
categories: ["vue3"]
---

## vue生产环境跨域不生效

### 开发环境跨域
项目集成了百度地图的api，在接入ip查询地址等功能时返回跨域报错，所以使用vite配置文件配置服务代理来解决。[传送门](https://blog.csdn.net/g18204746769/article/details/127115240)  

### 生成环境失效
测试跟我说内网功能失效了，在验证了程序确实build到最新版本之后，使用f12发现了api访问的地址并没有代理到百度地图api，而是在localhost下，看来是vite的代理失效了（vite在vue内容打包之后当然不存在）。


### 解决方法 
在nginx上面配置proxy_pass来代理
[传送门](https://www.cnblogs.com/Hijacku/p/15923670.html) 
[传送门](https://zhuanlan.zhihu.com/p/67058832) 
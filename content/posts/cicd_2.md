---
title: "gitlab ci/cd+ansible 远程ssh服务器deploy遇到的问题"
date: 2022-10-18T16:05:27+08:00
draft: false
categories: ["ci/cd","devops","ansible"]
---
1. 调用本地sh文件需要权限 
   1. 报错：/bin/bash: deploy.sh: Permission denied
   2. 解决：在yml调用sh前chmod777 deploy.sh
2. gitlab变量默认只在受保护的分支上生效
   1. 报错：直接使用变量会找不到
   2. 解决：[将对应分支设置为受保护](https://blog.csdn.net/halo1416/article/details/120198163)
3. ansible连接远程服务器失败
   1. 报错： Unable to parse /Users/mac/root@host1 as an inventory source
   2. 解决：注意：如果传入的节点只有一台，末尾的逗号不能省略[🔗](https://blog.csdn.net/liumiaocn/article/details/95456872)
      1. ansible all -i host161,host162 --list-hosts
      2. ansible all -i host161, --list-hosts
4. ssh密钥认证失败
   1. 报错：
        ```
            root@hosts | UNREACHABLE! => {
                "changed": false,
                "msg": "Failed to connect to the host via ssh: Host key verification failed.",
                "unreachable": true
            }
        ```
   2. 解决：需要参数 --ssh-common-args='-o StrictHostKeyChecking=no'
   3. 总体belike：ansible  --ssh-common-args='-o StrictHostKeyChecking=no'   -i "$DEPLOY_TO_HOSTS", "*" -m command -a 'ls'
5. ansible [参考](https://blog.csdn.net/qq_34556414/article/details/108668228)
   1. 新建文件夹，文件 用file模块
   2. 拷贝本机文件去远程服务器 用copy模块
   3. command模块 不能使用>,<,cd等方法，可以使用shell模块
6. "nohup &" linux后台运行程序
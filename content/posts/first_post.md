---
title: "上传github,ssh报错及解决方法"
date: 2022-08-30T11:11:27+08:00
draft: true
---


上传github报错
```
ssh: Could not resolve hostname github.com:HimawariSunFlower: nodename nor servname provided, or not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists
```

首先怀疑是公司gitlab账户覆盖掉github账户

ssh-add -l 确实没发现github的密钥

ssh -T git@github.com是通的，ping不通

尝试方法1 [hosts修改去可以ping通](https://blog.csdn.net/Thanksgining/article/details/104383480)，没有用


尝试方法2 [添加多个账户，添加到ssh-add](https://lujiahao0708.github.io/p/5c6f02af.html#:~:text=Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE%20%E9%80%9A%E5%B8%B8%E5%85%AC%E5%8F%B8%E4%BB%A3%E7%A0%81%E4%B8%80%E8%88%AC%E6%89%98%E7%AE%A1%E5%9C%A8%E5%85%AC%E5%8F%B8%E8%87%AA%E5%BB%BA%20Gitlab%20%E6%9C%8D%E5%8A%A1%E4%B8%8A%EF%BC%8C%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BB%A3%E7%A0%81%E6%89%98%E7%AE%A1%E5%9C%A8%20GitHub,Coding%20%E8%BF%99%E6%A0%B7%E7%9A%84%E7%BD%91%E7%AB%99%E4%B8%8A%E3%80%82%20Git%20%E8%B4%A6%E6%88%B7%E7%BB%8F%E5%B8%B8%E5%88%87%E6%8D%A2%E9%9D%9E%E5%B8%B8%E4%B8%8D%E6%96%B9%E4%BE%BF%EF%BC%8C%E8%BF%99%E5%B0%B1%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AA%20Git%20%E8%B4%A6%E6%88%B7%EF%BC%8C%E4%BB%A5%E5%90%91%E4%B8%8D%E5%90%8C%E7%9A%84%E7%BD%91%E7%AB%99%20push%20%E4%BB%A3%E7%A0%81%E3%80%82)，没有用，方法2延伸问题，ssh passphrase忘记，mac钥匙串可以找回


最终解决方法 [dns冲洗](https://stackoverflow.com/questions/9393409/ssh-could-not-resolve-hostname-github-com-name-or-service-not-known-fatal-th)
```
#Other ways to flush your dns, in windows in your terminal
ipconfig /flushdns

#on macos
dscacheutil -flushcache

#on linux
service nscd restart
```

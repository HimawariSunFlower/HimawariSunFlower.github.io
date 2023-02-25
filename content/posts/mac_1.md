---
title: "mac常用快捷键"
date: 2022-09-13T11:11:27+08:00
draft: false
categories: ["mac"]
---
通用
```
    Home键=Fn+左方向

    End键=Fn+右方向

    PageUP=Fn+上方向

    PageDOWN=Fn+下方向
```

 <br/>

终端  
```
    tab 完整的目录或文件名称
    tab+tab 显示可能的目录或文件名称补全列表
    command+D 双屏
    shift+command+D 取消双屏
    control+A 到行首
    control+E 到行尾
    control+U 行删除
    command+home 到顶部
    command+end 到底部
    command+pageup 上一页
    command+pagedown 下一页
```

<br/>

goland
```
    #部分是改建
    command+z 撤消
    control+T 重做 (撤消撤销)
    command+R 替换
    shift+command+R 全局替换
    command+f 搜索
    shift+command+f 全局搜索
    command+<- 到行首
    command+-> 到行尾
    shift+command+<- 选中到行首
    shift+command+-> 选中到行尾
    command+delete 行删除
    option+command+l 格式化文件
    option+shift+command+[ 从当前光标选中到代码块开始 (没选到最开始就多敲一下[)
    option+shift+command+] 从当前光标选中到代码块结束
```

chrome
```
    mac 强制刷新：command+shift+r
    mac 普通刷新：command+r
```

trem
```
    pwd 当前路径
    ps -ef | grep 进程名  查看进程号
    sudo lsof -i :端口  查看端口被哪个进程监听
    sudo lsof -nP -p 进程号 | grep LISTEN      查看进程监听的端口
    sudo lsof -nP | grep LISTEN | grep 进程号  查看进程监听的端口
    sudo lsof -nP | grep LISTEN | grep 端口号  查看监听端口的进程
    netstat -ntlp | grep 进程名  查看端口
```

linux
```
    查看对应端口进程信息
    netstat -tunlp | grep 端口号

    根据进程名获取系统进程信息
    ps aux | grep 程序名
```

copilot+mac+goland [url](https://docs.github.com/cn/copilot/getting-started-with-github-copilot/getting-started-with-github-copilot-in-a-jetbrains-ide)
```
    接受建议：Tab
    拒绝建议：Esc
    打开Copilot：Command+Shift+A  （会打开一个单独的面板，展示10个建议）
    下一条建议：Option + ]
    上一条建议：Option + [
    触发行内Copilot: Option + \ （Coplit还没有给出建议或者建议被拒绝了，希望手工触发它提供建议）
```
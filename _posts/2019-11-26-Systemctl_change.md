---
layout: post
title: 换系统啦!
date: 2019-11-26
tags: 博客   
---
在满足日常需求的前提下，系统还是要选择一款看着用着都顺心的。。。

电脑换了一块固态盘，顺便就把用得糟心的Ubuntu换了，对于不花时间去“定制”系统的国内用户来说，还是深度更友好。
上手体验堪比Windows了（真·一点儿不折腾）。

不过说，啥系统都有个毛病，自带源速度都太慢，还是得换成[清华](https://mirror.tuna.tsinghua.edu.cn/help/debian/)或者
[阿里](https://developer.aliyun.com/mirror)之类的源。
```shell script
vim /etc/apt/sources.list

sudo apt-get update
```
为啥说Deepin更友好呢，因为日常使用的玩意儿大多数应用商店里面都有了，Windows还得去软件官网呢，这儿直接给你整一堆了，下载安装一键完成，
要多爽有多爽。

上面说的其实也就针对下QQ和微信（之前在Ubuntu上又要整Wine又要找包，完事儿还有很多Bug只能勉强使用），某些软件个人还是喜欢自己下包安装。

比如Anaconda，官网一条龙服务，完事儿换个[源](https://mirror.tuna.tsinghua.edu.cn/help/pypi/)，美滋滋。
```shell script
vim ~/.condarc
vim ~/.pip/pip.conf
```
```text
# conda:
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud

# pip:
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```
环境是搭好了，自个儿下载的软件还得整个桌面图标，或者你愿意次次用命令行起也行。

举个栗子 `Pycharm.desktop`:
```text
[Desktop Entry]
Encoding=UTF-8
Exec="/home/jayzau/pycharm-2019.2.5/bin/pycharm.sh"
Icon=/home/jayzau/Pictures/icon/pycharm.png
Name=Snake
Name[zh_CN]=Spider-man
Type=Application
X-Deepin-Vendor=user-custom
```

工具都齐了，为了搞事情方便，`ssh`得用起来，这也是个人选用Linux的原因之一。自带终端太丑，强烈推荐贼好用的`Yakuake`+`Konsole`。

```shell script
# 生成
ssh-keygen -t rsa
# 查看
cat ~/.ssh/id_rsa.pub
# 推到服务器
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```

推荐搭配`alias`食用（自定义快捷命令）。例如常用命令`ll`实际上是定义了`alias ll='ls -l --color=auto'`，
取消自定义命令`unalias ll`。或直接写入`~/.bashrc`后刷新`source ~/.bashrc`。

---
一分钟后：

提交了主页居然不显示绿点，这尼玛谁能忍啊，哎，还是怪自己手贱填了一个不常用的邮箱。。。

```shell script
git config --global user.email 'github设置的email'
```

解决！

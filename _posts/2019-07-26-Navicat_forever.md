---
layout: post
title: Navicat premium伪破解实现"无限期"使用
date: 2019-07-26
tags: 博客   
---
Navicat作为一款数据库可视化工具，为工作带来了很大的便利，但毕竟是商业化软件，需要付费使用。

### 目标

实现永久免费使用Navicat工具

### 环境和工具

Ubuntu 18.04.2 LTS

Python 3.6.8

Navicat

### 原理

Navicat通过配置文件`user.reg`来读取试用过期时间，所以只要更改文件内容就可以实现“永不过期”。

Navicat在启动时读取不到`user.reg`会重新生成一个，试用期也会重置为14天。

用户数据也在此文件中，所以只能更改部分与过期时间相关的数据，否则就相当于完全重装Navicat了。

### 代码

因为单次试用只有14天，手动更改太麻烦，找到规律完全可以用程序代替。

```python
# -*- coding: utf-8 -*-
"""
@AUTH       Zau
@ADD TIME   2019/07/26
"""
import os
import re
import subprocess
import time


def matching(reg: str):

    classes = re.findall('.Software.*?Classes.*?Info] [0-9]{10}\n#time=.*?\n".*?"=".*?"', reg)
    premium = re.findall('.Software.*?PremiumSoft.*?Info] [0-9]{10}\n#time=.*?\n".*?"=".*?"', reg)

    return classes, premium


def replace(old: str, new: str) -> str:
    new_classes, new_premium = matching(new)
    old_classes, old_premium = matching(old)

    for i, string in enumerate(new_classes):
        old = old.replace(old_classes[i], string)
    for i, string in enumerate(new_premium):
        old = old.replace(old_premium[i], string)

    return old


def start_cat():
    command = '/opt/navicat/start_navicat'
    subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)


def kill_cat():
    command = "ps -ef | grep /opt/navicat/Navicat/Navicat.exe | grep -v grep | awk '{print $2}'"
    pid = subprocess.getoutput(command)
    subprocess.run(['kill', pid])


def run():
    file = '/home/zau/.navicat64/user.reg'
    file_bak = '/home/zau/.navicat64/user.reg.bak'
    """检查文件"""
    if not os.path.isfile(file):
        return
    if os.path.isfile(file_bak):
        os.remove(file_bak)
    """备份"""
    os.rename(file, file_bak)
    """重新生成文件"""
    start_cat()
    """检查文件是否已生成 生成完毕后可关闭"""
    timeout = 60
    while not os.path.isfile(file):
        time.sleep(1)
        timeout -= 1
        if not timeout:
            raise RuntimeError('生成配置文件超时，请手动打开Navicat生成！')
    kill_cat()
    """读文件"""
    with open(file, 'r') as f:
        new_file = f.read()
    with open(file_bak, 'r') as f:
        old_file = f.read()
    """换数据"""
    old_file = replace(old_file, new_file)
    """覆盖新文件 删除备份"""
    with open(file, 'w') as f:
        f.write(old_file)
    os.remove(file_bak)


if __name__ == '__main__':
    run()

```
最后将代码加入定时任务，例如每13天自动执行一次，就大功告成了！

---
layout: post
title: shell脚本操作远程服务器
date: 2019-12-04
tags: 博客   
---
### 基本方法

- **方法1**：

  ```shell
  echo "开始执行"
  ssh root@101.200.120.188 'echo "创建文件test"; mkdir test; echo "创建完毕，准备退出"; exit'
  echo "执行完毕"
  ```

  `ssh`登陆后，服务器内进行的操作要写到单引号内（`''`），不然会进入服务器终端操作界面。

  网上博客都说引号内指令不能换行，但是写一行太不美观，可读性也差。实践了一下，换行操作是可行的。

  ```shell
  echo "开始执行"
  ssh root@101.200.120.188 '
  echo "创建文件test"; 
  mkdir test; 
  echo "创建完毕，准备退出"; 
  exit'
  echo "执行完毕"
  ```

  两者执行效果相同。

- **方法2**：

  ```shell
  echo "开始执行"
  
  ssh root@101.200.120.188 << EOF
  
  echo "创建文件test"
  mkdir test
  echo "创建完毕，准备退出"
  exit
  
  EOF
  
  echo "执行完毕"
  ```

  利用`EOF`，看起来美观多了，实际效果也还不错。不过在`ssh`连接之后会打印一条本不是我们主动打印的信息：

  `Pseudo-terminal will not be allocated because stdin is not a terminal. 因为stdin不是终端，所以不会分配伪终端。`

  解决方案，增加-t -t参数来强制伪终端分配，即使标准输入不是终端。

  ```shell
  echo "开始执行"
  
  ssh -t -t root@101.200.120.188 << EOF
  mkdir test
  exit
  EOF
  
  echo "执行完毕"
  ```

  这样做还有一个好处，就是执行的命令以及响应的结果都会打印出来，就相当于在本机手动一条一条输入执行命令一样，更方便排查错误。

---

### 实例

实现自动拉取代码仓库代码。

##### 脚本：

```shell
ssh -t -t root@101.200.120.188 << EOF
cd PythonProjects
[ -d ./architect-awesome ] && echo "Found folder." || git clone git@github.com:jayzau/architect-awesome.git
cd architect-awesome
git pull
exit
EOF
```

##### 连续运行两次：

```shell
(base) jayzau@jayzau-PC:~/Documents$ ./demo.sh 
Last login: Wed Dec  4 14:25:11 2019 from 106.87.5.134

Welcome to Alibaba Cloud Elastic Compute Service !

cd PythonProjects
[ -d ./architect-awesome ] && echo "Found folder." || git clone git@github.com:jayzau/architect-awesome.git
cd architect-awesome
git pull
exit
(base) [root@jayzau ~]# cd PythonProjects
older." || git clone git@github.com:jayzau/architect-awesome.git&& echo "Found f 
正克隆到 'architect-awesome'...
remote: Enumerating objects: 789, done.
remote: Total 789 (delta 0), reused 0 (delta 0), pack-reused 789
接收对象中: 100% (789/789), 2.17 MiB | 34.00 KiB/s, done.
处理 delta 中: 100% (253/253), done.
(base) [root@jayzau PythonProjects]# cd architect-awesome
(base) [root@jayzau architect-awesome]# git pull
Already up-to-date.
(base) [root@jayzau architect-awesome]# exit
登出
Connection to 101.200.120.188 closed.
(base) jayzau@jayzau-PC:~/Documents$ ./demo.sh 
Last login: Wed Dec  4 14:25:53 2019 from 106.87.5.134

Welcome to Alibaba Cloud Elastic Compute Service !

cd PythonProjects
[ -d ./architect-awesome ] && echo "Found folder." || git clone git@github.com:jayzau/architect-awesome.git
cd architect-awesome
git pull
exit
(base) [root@jayzau ~]# cd PythonProjects
older." || git clone git@github.com:jayzau/architect-awesome.git&& echo "Found f 
Found folder.
(base) [root@jayzau PythonProjects]# cd architect-awesome
(base) [root@jayzau architect-awesome]# git pull
Already up-to-date.
(base) [root@jayzau architect-awesome]# exit
登出
Connection to 101.200.120.188 closed.
(base) jayzau@jayzau-PC:~/Documents$ 
```

可以看出程序是正常执行了，但是`older." || git clone git@github.com:jayzau/architect-awesome.git&& echo "Found f`这一部分还是让人很不爽。于是我又试了试第一种写法，运行结果如下：

```shell
(base) jayzau@jayzau-PC:~/Documents$ ./demo2.sh 
开始执行
正克隆到 'architect-awesome'...
Already up-to-date.
执行完毕
(base) jayzau@jayzau-PC:~/Documents$ ./demo2.sh 
开始执行
Found folder.
Already up-to-date.
执行完毕
(base) jayzau@jayzau-PC:~/Documents$ 
```

Awesome！打印界面瞬间清爽了好多！由于没有终端，所以克隆项目的时候无法看到进度，这可能算是一个扣分项，但是只要代码逻辑理清楚了，保证程序不出错的情况下，还是更喜欢清爽的写法1。

---

什么是EOF：

> **EOF**是一个计算机术语，为**End Of File**的缩写，在操作系统中表示资料源无更多的资料可读取。 资料源通常称为档案或串流。 通常在文本的最后存在此字符表示资料结束。    ——百度百科

[EOF是什么？ - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/11/eof.html)

[linux shell脚本EOF妙用- zongshi1992的博客- CSDN博客](https://blog.csdn.net/zongshi1992/article/details/71693045)


---
layout: post
title: 利用谷歌浏览器插件获得更好的浏览体验
date: 2019-11-30
tags: 博客   
---

### 目的

绕过一些前端做出的浏览限制，让原本“需要授权”的内容正常展现出来。

例如：

- 需要登录才能看所有菜单的大众点评

![](/images/posts/Chrome_tools/dzdp.png)

- 需要关注公众号才能浏览的 -> [静觅 崔庆才的个人博客](https://cuiqingcai.com/)

![](/images/posts/Chrome_tools/cqc.png)


### 动机

对于一些网站来说，平日里很少会用到，注册账号又需要绑定手机号之类的信息太麻烦，所以就另辟蹊径希望绕过一些要求来实现自己的需求。
对于崔哥这种网站，其实我是关注了公众号的，但是由于cookie的问题，还是得经常验证。程序员都是懒人，微信扫来扫去确实还是比较烦，
只好对不起崔哥想要的流量了。

### 实现

其实按F12检查一下就能发现，页面内容是一次性加载好了的，只是被隐藏了而已。

![](/images/posts/Chrome_tools/cqc1.png)

反过来，我们只需要将对应的css样式调整一下就ok。

![](/images/posts/Chrome_tools/cqc2.png)

最后利用油猴脚本，解放双手～

![](/images/posts/Chrome_tools/cqc3.png)

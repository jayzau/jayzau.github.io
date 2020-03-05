---
layout: post
title: 容器化Nginx实现端口转发
date: 2019-03-05
tags: 博客   
---

### Docker
> Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

[Docker安装(Ubuntu)](https://www.runoob.com/docker/ubuntu-docker-install.html)

[Docker安装(CentOS)](https://www.runoob.com/docker/centos-docker-install.html)

---

### Nginx
> Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

#### 利用Docker安装Nginx

搜索版本
```shell script
docker search nginx
```

下载镜像
```shell script
docker pull nginx:latest
```

创建容器并运行
```shell script
docker run --name mynginx -p 8080:80  -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf  -v /etc/nginx/log:/var/log/nginx -d  --restart=always docker.io/nginx
```

运行成功后打开8080端口就能看见Nginx欢迎页面。


---

### 实际运用

做爬虫可能会需要用到splash来抓取数据，一个splash服务承载力有限，开多个splash服务意味着要开多个端口，所以利用Nginx做个端口转发，还能实现负载均衡。

流程：访问主机8080端口->容器内80端口->splash服务端口1 or 2 or 3

#### 第一步：找到Docker对应的地址
这里不能够直接写主机地址或者127.0.0.1，主机地址需要额外开放端口，127则会映射到容器内对应端口。
输入
```shell script
ifconfig
```
找到docker那一栏，inet后的地址就是我们需要的。

#### 第二步：更改Nginx配置
```shell script
vim /etc/nginx/nginx.conf
```
修改以下部分即可。`172.17.0.1`为第一步找到的inet地址，端口号为由Docker启动的splash服务映射的端口。weight表示权重。
```text
 upstream spalsh {
            server 172.17.0.1:50000 weight=5;
            server 172.17.0.1:50001 weight=5;
            server 172.17.0.1:50002 weight=5;
            server 172.17.0.1:50003 weight=5;
            server 172.17.0.1:50004 weight=5;
            server 172.17.0.1:50005 weight=5;
            server 172.17.0.1:50006 weight=5;
    }

    server{ 
            listen 8080; 
                    server_name spalsh; 

            location / { 
                    proxy_pass         http://spalsh; 
                    proxy_connect_timeout 100s;
                    proxy_read_timeout 100s;
                    proxy_send_timeout 100s;
            #       proxy_set_header   Host             $host; 
            #       proxy_set_header   X-Real-IP        $remote_addr; 
            #       proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
            } 
    }
```

#### 第三步：重启Nginx容器
使配置生效。


这样一来，爬虫只需要访问固定地址，便可以自动转到不同splash服务上，提高并发量。

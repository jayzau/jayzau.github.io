---
layout: post
title: PHP环境安装
date: 2020-09-28
tags: 备忘录   
---

公司发展需要，开始学习PHP。

### 安装环境过程

- *安装依赖*

这一项很重要，最开始我准备编译的时候再根据报错来选择性安装依赖，试过之后发现是真的麻烦。还是一步到位吧。

```shell
apt-get install gcc make openssl curl libbz2-dev libxml2-dev libjpeg-dev libpng-dev libfreetype6-dev libzip-dev libssl-dev libmcrypt-dev autoconf automake m4 libsqlite3-dev libonig-dev
```

- [官网下载需要的源码](https://www.php.net/downloads)

- [编译安装](https://www.php.net/manual/zh/install.unix.php)

[参数选项在这](https://www.php.net/manual/zh/configure.about.php)，根据需要的选项配置。

```shell
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --enable-mbstring --with-openssl --enable-ftp --with-gd --with-jpeg-dir=/usr --with-png-dir=/usr --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pear --enable-sockets --with-freetype-dir=/usr --with-zlib --with-libxml-dir=/usr --with-xmlrpc --enable-zip --enable-fpm --enable-xml --enable-sockets --with-gd --with-zlib --with-iconv --enable-zip --with-freetype-dir=/usr/lib/ --enable-soap --enable-pcntl --enable-cli --with-curl
```

我是在网上Copy的，应该比较通用吧。编译过程中可能会报一些依赖包的错，根据报错装上对应依赖再重新编译就行了。

```shell
// configure: error: Cannot find OpenSSL's <evp.h>
sudo apt-get install -y autoconf g++ make openssl libssl-dev libcurl4-openssl-dev
sudo apt-get install -y libcurl4-openssl-dev pkg-config
sudo apt-get install -y libsasl2-dev
// configure: error: jpeglib.h not found.
sudo apt-get install libjpeg-dev
```

以上是错误示例。

成功编译后会感谢你使用PHP。

> Thank you for using PHP.

- *make && make install*

- 安装完成

这时候试试`/usr/local/php/bin/php -v`应该就会打印出来版本信息。

想在任何路径都能调用php，加个软链。

```shell
ln -s /usr/local/php/bin/php /usr/bin/php
```

### php-fpm

暂时没弄懂是干嘛的。

[朋友php7.4的安装过程](https://express.iefoam.com/detail?id=551&uId=zBRIkgKKSsGNMwtU8VriLVmpn2gdutzJ)带得有，以后用到了再写吧。


---

### 后续

**关于`php-fpm`**

个人理解`php-fpm`就是用来解析`php`脚本的。不做相关配置的话，`php`文件就仅仅只是个普通文件。

**关于扩展安装**

在编译安装`php`的参数选项步骤的时候就可以安装部分扩展。但是像`xdebug`扩展就不行，需要手动安装配置。

[xdebug安装](https://xdebug.org/wizard)

![xdebug安装过程](/images/posts/PHPInstall/xdebug_install.png)

其实扩展安装都差不多，下包，解包，编译，安装，最后写入配置。

不同的扩展写入的配置不太一样，比如`xdebug`最后写入的是`zend_extension = /usr/local/php/lib/php/extensions/no-debug-zts-20190902/xdebug.so`，而`redis`写入的是`extension = /usr/local/php/lib/php/extensions/no-debug-zts-20190902/redis.so`。

变量名不知道有什么规律，看文档最好。后面指向的`so`文件路径一定要正确就是了。


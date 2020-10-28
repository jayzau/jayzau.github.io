---
layout: post
title: PHP的一些扩展
date: 2020-10-28
tags: 备忘录   
---

---

### XDebug 配置

**前提是已经正常配置好PHP+Apache/Nginx,能从浏览器正常访问**

**IDE为PHPStorm**

复制`phpinfo()`页面信息到[xdebug安装](https://xdebug.org/wizard).

找到`php.ini`写入配置信息.

```shell
zend_extension = /usr/local/php/lib/php/extensions/no-debug-zts-20190902/xdebug.so
;开启远程调试
xdebug.remote_enable = On
;客户机ip
xdebug.remote_host="127.0.0.1"
;客户机xdebug监听端口和调试协议
xdebug.remote_port=9001
xdebug.remote_handler=dbgp
;idekey 区分大小写
xdebug.idekey="PHPSTORM"
xdebug.profiler_enable = off
xdebug.profiler_enable_trigger = off
xdebug.profiler_output_name = cachegrind.out.%t.%p
;idekey 区分大小写
;xdebug.profiler_output_dir = ""
```

随后用PHPStorm打开需要调试的项目.

- 找到`File->Settings->Languages & Frameworks->PHP->Debug`页面,将`Xdebug`标签下的`Debug port`改为上述配置中`xdebug.remote_port`对应的值.

- 找到`File->Settings->Languages & Frameworks->PHP->Debug->DBGp Proxy`页面,`IDE key`/`Host`/`Port`分别对应上述配置中的`xdebug.idekey`/`xdebug.remote_host`/`xdebug.remote_port`.

- 找到`File->Settings->Languages & Frameworks->PHP->Servers`页面,添加一条记录值,`Host`对应`xdebug.remote_host`,`Port`选择`Apache/Nginx`对应端口,`Debugger`选择`Xdebug`,`Name`随意.

- 找到`File->Settings->Languages & Frameworks->PHP->Debug`页面,`Pre-configuration`标签下有一个`Validate`可以点击查看已完成的配置(如果成功).

- 找到`Run->Edit Configurations`页面,新增一条`PHP Web Page`配置,`Server`选择第三步添加的记录值,`Start URL`写入浏览器正常访问该项目的链接,`Name`随意.

- 打开监听:`Run->Start Listening for PHP Debug Connections`.

- 最后就可以像`Python`一样打断点`Debug`了.

---

### Blade Filters

**基于`Laravel`框架的前端Blade模板,实现过滤器语法.**

Github直达: [blade-filters](https://github.com/conedevelopment/blade-filters)

安装:`composer require thepinecode/blade-filters`

使用: 

- 生成一个新的[服务提供者](https://learnku.com/docs/laravel/8.x/providers/9362): `php artisan make:provider BladeFilterServiceProvider`,直接用自带的`AppServiceProvider`也行.

- 在引导方法内编写自定义过滤器:

```php
/**
 * Bootstrap services.
 *
 * @return void
 */
public function boot()
{
    // 时间戳转自定义格式时间字符
    BladeFilters::macro('dateStr', function ($timestamp, $format='Y-m-d H:i:s'){
        return date($format, $timestamp);
    });
}
```

- 注册服务提供者(如果是新生成的话):`config/app.php`文件内找到`providers`数组,将文件路径写入.

- 在模板内使用:

```djangotemplate
<th>{{ 1600000000 | dateStr }}</th>
<th>{{ 1600000000 | dateStr:'Y-m-d' }}</th>
```



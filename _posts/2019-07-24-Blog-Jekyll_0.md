---
layout: post
title: Jekyll使用模板
date: 2019-07-24 
tags: 博客   
---
### 安装Jekyll

[Jekyll中文文档](http://jekyll.bootcss.com/)

### 模板下载

[免费](http://jekyllthemes.org/)

[部分收费](https://jekyllthemes.io/)

安装教程不多记录，网上很多教程。

[本站模板作者教你一步步搭建环境](http://baixin.io:8000/2016/10/jekyll_tutorials1/)

----

### 使用模板启动服务报过的错

```
⇒  jekyll serve
Traceback (most recent call last):
	5: from /usr/local/bin/jekyll:23:in `<main>'
	4: from /usr/local/bin/jekyll:23:in `load'
	3: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/exe/jekyll:11:in `<top (required)>'
	2: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/lib/jekyll/plugin_manager.rb:48:in `require_from_bundler'
	1: from /usr/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require'
/usr/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require': cannot load such file -- bundler (LoadError)
```
加载出了问题，查看一下`bundler`版本信息：
```
⇒  bundler --version
Traceback (most recent call last):
	2: from /usr/local/bin/bundler:23:in `<main>'
	1: from /usr/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
/usr/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundler (Gem::GemNotFoundException)
```
还是报错，解决参考文章↓↓↓

[How to Solve Error can’t find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)](http://www.dark-hamster.com/application/how-to-solve-error-cant-find-gem-bundler-0-a-with-executable-bundle-gemgemnotfoundexception/)

```
⇒  gem list bundle

*** LOCAL GEMS ***

bundle (0.0.1)
bundler (2.0.2)


⇒  vim Gemfile.lock

BUNDLED WITH
   1.16.1

# 改为

BUNDLED WITH
   2.0.2
```
```
⇒  bundler --version
Bundler version 2.0.2
```
解决完问题后尝试重启依然报错：
```
⇒  jekyll serve     
Traceback (most recent call last):
	12: from /usr/local/bin/jekyll:23:in `<main>'
	11: from /usr/local/bin/jekyll:23:in `load'
	10: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/exe/jekyll:11:in `<top (required)>'
	 9: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/lib/jekyll/plugin_manager.rb:50:in `require_from_bundler'
	 8: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler.rb:107:in `setup'
	 7: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:20:in `setup'
	 6: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:108:in `block in definition_method'
	 5: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/definition.rb:226:in `requested_specs'
	 4: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/definition.rb:237:in `specs_for'
	 3: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/definition.rb:170:in `specs'
	 2: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:81:in `materialize'
	 1: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:81:in `map!'
/var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:87:in `block in materialize': Could not find public_suffix-3.0.3 in any of the sources (Bundler::GemNotFound)
```
看样子是缺包了，安装一下：
```
⇒  bundle install
```
安装完毕之后，再次启动报错：
```
⇒  jekyll serve  
Traceback (most recent call last):
	10: from /usr/local/bin/jekyll:23:in `<main>'
	 9: from /usr/local/bin/jekyll:23:in `load'
	 8: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/exe/jekyll:11:in `<top (required)>'
	 7: from /var/lib/gems/2.5.0/gems/jekyll-3.8.6/lib/jekyll/plugin_manager.rb:50:in `require_from_bundler'
	 6: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler.rb:107:in `setup'
	 5: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:26:in `setup'
	 4: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:26:in `map'
	 3: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:148:in `each'
	 2: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/spec_set.rb:148:in `each'
	 1: from /var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:31:in `block in setup'
/var/lib/gems/2.5.0/gems/bundler-2.0.2/lib/bundler/runtime.rb:319:in `check_for_activated_spec!': You have already activated public_suffix 3.1.1, but your Gemfile requires public_suffix 3.0.3. Prepending `bundle exec` to your command may solve this. (Gem::LoadError)
```
看着像是包版本不对，两种解决方案：

- 修改`Gemfile.lock`文件，将对应的版本号更改为上述版本号

- 卸载多余的版本，只保留一个



```
⇒  sudo gem uninstall jekyll-watch

Select gem to uninstall:
 1. jekyll-watch-2.1.2
 2. jekyll-watch-2.2.1
 3. All versions
> 2
Successfully uninstalled jekyll-watch-2.2.1
```
全部更改完毕后就能成功启动了！

---
layout: post
title: laravel框架一些不顺手的地方
date: 2020-10-29
tags: 博客   
---

个人吐槽.

### Model相关

1. **`DB`方式查询和`Model(Eloquent ORM)`方式查询返回数据类型不一致**

都是官方工具类,并且`Model`查询方法兼容`DB`查询方法([查询构造器](https://learnku.com/docs/laravel/8.x/queries/9401)),
那么按道理查询结果返回的数据格式应该一致.但是!`DB`的`toArray()`方法返回的数据中,只能将最外层转换为数组,其内部还是`stdClass`对象.而`Model`的`toArray()`全部转换为数组.
最初学习的时候为了方便,全部使用的`DB`方式查询,后来更换为`Model`方式后,代码从`Controller`改到`View`...

2. **IDE支持不友好**

其实也还好,就是`DB`改为`Model`后,PHPStorm代码提示报黄了,一度认为是我写法不对,但是官方文档也这样写啊...后来查了帖子,两种解决方案.

第一种,装一个插件,[laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper),好像很麻烦,还得改项目配置.

第二种,查询时加一个`query()`,后面再跟常规查询语句.`Model::query()->where()...`.挺喜欢这种写法的.

3. **服务太周到了**(这也是错?)

其实个人觉得`laravel`的模型层很棒,使用很方便.但是不明白为什么它要主动去维护一些东西.比如:

> Eloquent 也会假设每个数据表都有一个名为 `id` 的主键列。你可以定义一个受保护的 `$primaryKey` 属性来重写约定。
>
> 默认情况下，Eloquent 预期你的数据表中存在 `created_at` 和 `updated_at` 两个字段 。如果你不想让 Eloquent 自动管理这两个列， 请将模型中的 `$timestamps` 属性设置为 false
>
> ...

这些东西难道不应该是`laravel`可以主动提供,但是否使用由用户来决定吗?对于这点,还是更喜欢`Flask`的理念.



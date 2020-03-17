---
layout: post
title: rq-dashboard代码优化
date: 2020-03-17
tags: 博客   
---

在使用`rq`队列处理异步任务的时候，总会希望有一个地方能够实时看见任务进度，执行结果。

[rq-dashboard](https://pypi.org/project/rq-dashboard/)很好的实现了这个功能，而且上手简单，功能齐全，界面美观。

不过在使用过程中遇见一个问题，如下图。

![](/images/posts/Rq_dashboard/screenshot.png)


可以看见，`rq-dashboard`非常贴心的提供了失败任务专栏。报错的任务全都会在这下面展现，报错详情，任务id，应有尽有，右上侧还提供了`Requeue All`方法，可以让失败的任务一键归队重新执行。

不过由于一些未知的原因，我的部分任务在执行过程中丢失了信息，队列里红色的`failed job`异常显眼，但详情栏里并没有找到对应的`job`。出现这种情况的时候一键归队也失效，响应码500，看来是后端代码出了问题。

`rq-dashboard`使用的是`flask`框架，轻量级，代码比较直观，调试中不难发现`Requeue All`方法是遍历所有失败任务，通过任务id一个一个一次归队，那么只要其中任何一个任务获取失败（NoSuchJobError），就会导致后续代码不能正常进行。

```python
# rq-dashboard 视图层函数

@blueprint.route('/requeue-all', methods=['GET', 'POST'])
@jsonify
def requeue_all():
    fq = get_failed_queue()
    job_ids = fq.job_ids
    count = len(job_ids)
    for job_id in job_ids:      # 遍历失败队列 获取任务id
        requeue_job(job_id, connection=current_app.redis_conn)
    return dict(status='OK', count=count)

```

```python
# rq  job.py  453行

    # Persistence
    def refresh(self):  # noqa
        """Overwrite the current instance's properties with the values in the
        corresponding Redis key.

        Will raise a NoSuchJobError if no corresponding Redis key exists.
        """
        data = self.connection.hgetall(self.key)
        if not data:
            raise NoSuchJobError('No such job: {0}'.format(self.key))       # 任务获取失败
        self.restore(data)
```

找到了原因，就容易解决了。

只需要捕捉到异常任务，然后再删除相应数据就行了，正常任务按原方法处理。

三方包代码不建议直接修改，所以新建了个入口文件，再替换相关方法。代码如下：

```python
"""
RQ-dashboard version: 0.6.1
Python RQ version: 1.1.0
"""
import importlib
import os

from rq import requeue_job, Queue
from rq.exceptions import NoSuchJobError
from rq.job import Job
from rq.registry import FailedJobRegistry
from rq_dashboard import cli
from rq_dashboard.web import jsonify, current_app
from rq_dashboard.compat import get_failed_queue


@cli.blueprint.route('/requeue-all', methods=['GET', 'POST'])
@jsonify
def own_requeue_all():
    fq = get_failed_queue()
    job_ids = fq.job_ids
    count = len(job_ids)
    reg_job_ids = []
    for job_id in job_ids:
        try:
            requeue_job(job_id, connection=current_app.redis_conn)
        except NoSuchJobError:      # 异常捕捉
            job = Job(id=job_id)
            job.connection.delete(job.key)      # 删除相关数据
            reg_job_ids.append(job_id)
    if reg_job_ids:     # 删除相关数据
        for n in Queue.all():
            name = n.name
            br = FailedJobRegistry(name=name)
            r = br.connection.zrem(br.key, *job_ids)
    return dict(status='OK', count=count)


def make_flask_app(config, username, password, url_prefix,
                   compatibility_mode=True):
    """Return Flask app with default configuration and registered blueprint."""
    app = cli.Flask(cli.__name__)       # 全都用原有属性

    # Start configuration with our built in defaults.
    app.config.from_object(cli.default_settings)

    # Override with any settings in config file, if given.
    if config:
        app.config.from_object(importlib.import_module(config))

    # Override from a configuration file in the env variable, if present.
    if 'RQ_DASHBOARD_SETTINGS' in os.environ:
        app.config.from_envvar('RQ_DASHBOARD_SETTINGS')

    # Optionally add basic auth to blueprint and register with app.
    if username:
        cli.add_basic_auth(cli.blueprint, username, password)
    app.register_blueprint(cli.blueprint, url_prefix=url_prefix)

    # 由于路由一样，重写的方法比原有方法后注册，所以默认会调用原有方法，这里逆个序解决
    app.url_map._rules.reverse()
    return app


if __name__ == '__main__':
    # 利用猴子补丁 替换原有 make_flask_app 方法
    cli.make_flask_app = make_flask_app
    # 原有方法调用
    cli.main()

```

![](/images/posts/Rq_dashboard/screenshot.png)


## 舒服了。

---

[猴子补丁的含义是指在动态语言中，不去改变源码而对功能进行追加和变更。](https://zhidao.baidu.com/question/1449368139271751780.html)

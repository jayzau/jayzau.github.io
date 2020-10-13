---
layout: post
title: MongoDB角色权限
date: 2020-10-13
tags: 备忘录   
---

> Read：允许用户读取指定数据库
> 
> readWrite：允许用户读写指定数据库
> 
> dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
> 
> userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
> 
> clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
> 
> readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
> 
> readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
> 
> userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
> 
> dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
> 
> root：只在admin数据库中可用。超级账号，超级权限。

---

登入mongo，默认本地27017端口，管理员登入不需要加`authenticationDatabase`。

```shell
> mongo -u "user" -p "pwd" --port 27017 --host localhost --authenticationDatabase "database"
```

选择`db`，没有`db`会新建，新建的`db`没有`collection`不会显示。然后创建用户，给权限。

```shell
> use database

> db.createUser({user:"jayzau",pwd:"123456",roles:[{role:"dbAdmin",db:"database"},{...}]})
Successfully added user: {
        "user" : "jayzau",
        "roles" : [
                {
                        "role" : "dbAdmin",
                        "db" : "database"
                }
        ]
}

> db.auth(“jayzau”,”123456”)
```

...`dbAdmin`并不包括读写权限，所以需要修改。`.getUser`方法可以查看角色信息。

```shell
{
        "_id" : "database.jayzau",
        "userId" : UUID("2ee43f49-a793-4d59-88e8-cb012c29d4e4"),
        "user" : "jayzau",
        "db" : "database",
        "roles" : [
                {
                        "role" : "dbAdmin",
                        "db" : "database"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```

`mechanisms`是验证方式，和版本有关。3.0版本之前貌似是`MONGODB-CR`。`mongo --help`能看见`--authenticationMechanism`参数。写错了也连不上。

`.updateUser`方法里的`roles`参数是覆盖，而不是`update`。所以权限得写全。

```shell
> db.updateUser("jayzau", {roles:[{role:"readWrite", db:"database"},{role:"dbAdmin",db:"database"}]})
```

这样就用`jayzau`用户在`database`里面随意造作了！

title: mongodb配置及设置密码
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-11-14 21:11:09
description: 腾讯云不在支持centos6.5了，于是重装了服务器，配一下mongo
photos: http://www.runoob.com/wp-content/uploads/2013/10/mongodb-logo.png
---

## balabala
腾讯云不在支持centos6.5了，于是重装了服务器，配一下mongo。

1、首先mongodb的用户名和密码是基于特定数据库的，而不是基于整个系统的。所有数据库都需要设置密码。不像mysql用户密码适用于所有数据库。

2、mongodb安装后自身没有密码会有安全问题，会很容易被勒索
<img src="https://image-static.segmentfault.com/128/268/1282680596-59e16f90f0c27_articlex">

## 过程
1、show dbs;
> 找admin数据库。新版mongodb没有admin数据库，没事use时会自动创建。

2、use admin;
> 进入admin数据库

3、创建管理员账户
```javascript
db.create({
  user: "用户名",
  pwd: "密码",
  role: [{
    role: "userAdminAnyDatabase",
    db: "admin"
  }]
})
```
> mongodb中用户是基于身份role的，该管理员账户的role是userAdminAnyDatabase。其中userAdmin代表用户管理身份，AnyDatabase代表可以管理任何数据库

4、验证是否添加成功用户是否添加成功
> db.auth("用户名", "密码")  如果返回1则表示成功
> exit退出系统。可以把db.auth()方法理解为用户的验证功能

5、修改配置
> sudo vi /etc/mongod.conf
> 找到#security: 取消注释，修改为：
> security:
> authorization: enabled #注意缩进，缩进参照配置文件其他配置。缩进错误可能造成重启不成功。

6、重启mongodb，sudo service mongod restart

7、进入mongodb，用管理员账户登录，用该账户创建其他数据库管理员账号
> use admin;
> db.auth("用户名", "密码")；

8、新建需要管理的mongodb数据的账号密码
> use yourdatabase;
```javascript
db.createUser({ 
  user: "新的用户名",
  pwd: "新的密码", 
  roles: [{ 
    role: "dbOwner",
    db: "yourdatabase"
  }] 
})
```
> rote:dbOwner 代表数据库所有者角色，拥有最高该数据库最高权限。比如新建索引等

9、新建数据库读写账户
> use yourdatabase
```javascript
db.createUser({ 
  user: "youruser2", 
  pwd: "yourpassword2", 
  roles: [{ 
    role: "readWrite", 
    db: "yourdatabase" 
  }] 
})
```

10、
> 现在数据的用户名和密码就建好了。
> 可以使用：mongodb://youruser2:yourpassword2@localhost/yourdatabase来链接
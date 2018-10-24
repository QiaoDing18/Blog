title: Nginx配置小结
tags:
  - 计算机网络
categories:
  - net
author: 乔丁
date: 2018-10-24 16:09:09
description: nginx配置踩坑
photos: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1540380251097&di=3e3041dade1c342d383b361f1f0aca40&imgtype=0&src=http%3A%2F%2Fww2.sinaimg.cn%2Forj360%2F795bf814gw1f5qyfy82ayj208c08cgm1.jpg
---

## Balabal
想配置一下https，发现问题很多，小结一下

## 配置HTTPS遇到的问题

### Q1：nginx: \[emerg\] socket() [::]:80 failed (97: Address family not supported by protocol) 
其实网上解决办法写的很明确了，将一个文件中的
```
listen       80 default_server;
listen       [::]:80 default_server;

修改为:
listen       80;
#listen       [::]:80 default_server;
```
问题是这修改的是哪个文件？

出现这个问题后的nginx配置文件根目录下的nginx.conf是没有这个问题的，但是修改了/conf.d/default.conf文件中的这个就好了。

这里需要留意一下nginx装好后初始的配置中引入了哪些文件，有这么一句
```
include /etc/nginx/conf.d/*.conf;
```
这句之后开始配置的server{}，在conf.d目录下有这些文件：default.conf、ssl.conf、virtual.conf（其他俩分别是443端口、8000端口的默认配置，一般被直接注释掉了）

首先说明一下nginx根目录下的nginx.conf、conf.d文件夹、default.d文件夹的优先级：nginx.conf>conf.d 文件夹>default.d 文件夹。 default文件夹是备份文件，一般没啥用。但是如果在conf.d文件夹下设置了和根目录下相同的东西，比如这个问题里，有一个default_server在listen着80端口了，再次配置就会报错。

还有呢，nginx编写的指令顺序一般情况下没那么重要，要取决于nginx内部的解析阶段、机制。这里主要体现在跳转location上：
- 普通前端匹配的路径，例如 location / {}
- 抢占式前缀匹配的路径，例如 location ^~ / {}
- 精确匹配的路径，例如 location = / {}
- 命名路径，比如 location @a {}
- 无名路径，比如 if {}或者 limit_except {}生成的路径

值得一提的是，可以把每一个站点作为一个单独的配置文件会更方便维护。

### Q2：nginx: \[emerg\] a duplicate default server for 0.0.0.0:80 in /etc/nginx/
这个问题同上

### Q3：nginx: \[warn\] conflicting server name "tjoe18.cn" on 0.0.0.0:80, ignore..
这个问题是server_name写域名的时候没写www，我写的tjoe18.cn，改成www.tjoe18.cn就好了。那么这有啥区别呢：

tjoe18是域名，.cn是域名后缀，完整的域名是tjoe18.cn，也就是我花钱买的那个。

**说法一：**
www二级域名，tjoe18是一级域名。或者说就像join.xiyoumoble.com一样，这里的join就如同www了。
**说法二：**
www是主机名，cn是顶级域名，tjoe18.cn是二级域名，在二级域名下找名为www的主机

有两种说法我确认一下吧。暂时这么多
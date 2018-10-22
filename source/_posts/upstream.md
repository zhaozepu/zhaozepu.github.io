---
layout: nginx
title: Nginx upstream 负载均衡
date: 2018-10-12 17:10:21
header-img: "http://oab3kr3oh.bkt.clouddn.com/WechatIMG17.jpeg"
tags: nginx
---

Nginx 的 HttpUpstreamModule 提供对后端(backend)服务器的简单负载均衡。

1.新建3台vagrant虚拟机 A（192.168.33.10）、B（192.168.33.20）、C（192.168.33.30）
2.本地配置host
192.168.33.10  z.com
3.A 配置upstream模块
3.1 打开nginx.conf 在 http{}标签内配置upstream模块（以下是权重模式）

```php
upstream z.com {
                server 192.168.33.20:80 weight=1;
                server 192.168.33.30:80 weight=5;
        }
```

3.2 配置server
```php

server {
        listen 80;
        server_name a.com;
        access_log  logs/a.access.log main;
        location / {#通用规则
           proxy_pass http://a.com;#配置代理
        }
}
```

3.3 分别在B、C虚拟机中配置server
```php

server {
        listen 80;
        server_name a.com;
        access_log  logs/a.access.log main;

        location / {
                root /home/work/test;
                index index.html index.php;

                }
        }
```
        

3.4 在B、C中的/home/work/test目录下新建index.html文件，并输出不同内容（为了确认当前访问的是哪台机器）
3.5 重启3台虚拟机的nginx

浏览器访问z.com可以看到A服务器会按照配置的权重比轮询B、C两台服务器
为了测试宕机情况，手动将B服务器关机
shutdown -h now
继续访问z.com 会发现自动剔除了B服务器，每次访问都会命中C。


                

一、分配方式 
Nginx的upstream支持5种分配方式
1、轮询 轮询是upstream的默认分配方式,即每个请求按照时间顺序轮流分配到不同的后端服务器,如果某个后端服务器down掉后,能自动剔除。
```php

upstream backend { 
server 192.168.1.101:8888; 
server 192.168.1.102:8888; 
server 192.168.1.103:8888; 
}
```
 

2、weight 轮询的加强版,即可以指定轮询比率,weight和访问几率成正比,主要应用于后端服务器异质的场景下。

```php
upstream backend { 
server 192.168.1.101 weight=1; 
server 192.168.1.102 weight=2; 
server 192.168.1.103 weight=3; 
} 
```

3、ip_hash 每个请求按照访问ip(即Nginx的前置服务器或者客户端IP)的hash结果分配,这样每个访客会固定访问一个后端服务器,可以解决session一致问题。

```php
upstream backend { 
ip_hash; 
server 192.168.1.101:7777; 
server 192.168.1.102:8888 weight=2; 
server 192.168.1.103:9999 backup; 
} 
```

注意:1、当负载调度算法为ip_hash时,后端服务器在负载均衡调度中的状态不能是weight和backup。2、导致负载不均衡。
4、fair fair顾名思义,公平地按照后端服务器的响应时间(rt)来分配请求,响应时间短即rt小的后端服务器优先分配请求。如果需要使用这种调度算法,必须下载Nginx的upstr_fair模块。

```php
upstream backend { 
server 192.168.1.101; 
server 192.168.1.102; 
server 192.168.1.103; fair; 
} 
```

5、url_hash,目前用consistent_hash替代url_hash与ip_hash类似,但是按照访问url的hash结果来分配请求,使得每个url定向到同一个后端服务器,主要应用于后端服务器为缓存时的场景下。
```php

upstream backend { 
server 192.168.1.101; 
server 192.168.1.102; 
server 192.168.1.103; 
hash $request_uri; 
hash_method crc32; 
} 
```

其中,hash_method为使用的hash算法,需要注意的是:此时,server语句中不能加weight等参数。
提示:url_hash用途cache服务业务,memcached,squid,varnish。特点:每个rs都是不同的。



参数说明:
server address parameters
server:关键字,必选。
address:主机名、域名、ip或unix socket,也可以指定端口号,必选。
parameters:可选参数,可选参数如下:
1.down:表示当前server已停用
2.backup:表示当前server是备用服务器,只有其它非backup后端服务器都挂掉了或者很忙才会分配到请求。
3.weight:表示当前server负载权重,权重越大被请求几率越大。默认是1.
4.max_fails和fail_timeout一般会关联使用,如果某台server在fail_timeout时间内出现了max_fails次连接失败,那么Nginx会认为其已经挂掉了,从而在fail_timeout时间内不再去请求它,fail_timeout默认是10s,max_fails默认是1,即默认情况是只要发生错误就认为服务器挂掉了,如果将max_fails设置为0,则表示取消这项检查。
```php

upstream backend { 
server backend1.example.com weight=5; 
server 127.0.0.1:8080 max_fails=3 fail_timeout=30s; 
server unix:/tmp/backend3; 
} 

```

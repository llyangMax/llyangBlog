---
published: true
layout: post
title: Center OS 服务器搭建 Shadowsocks 服务
category: 其他
tags: 
  - VPS
  - VPN
  - shadowsock
time: 2017.10.20 15:16:00
excerpt: ShadowSocks（中文名影梭） 是由@clowwindy所开发的一个开源 Socks5 代理。在国内，由于GFW的存在，更多的使用者用它来上网。下面来记录一下在Center OS服务器上搭建Shadowsocks 服务的过程
---
###1. ShadowSocks 的安装
首先是选择一款合适的国外主机作为服务器，一般而言，用来作 ShadowSocks 的服务器的话，并不需要一台独立的国外主机，只需要选择一款虚拟主机（Virtual Private Server，简称 VPS）即可满足需要。比较常用的有BandwagonHOST（俗称搬瓦工）、 Digital Ocean、 Vultr、Linode。我最初选择的是搬瓦工后来不知道什么原因ip被墙了，就换成了[Vultr](https://www.vultr.com/?ref=7229048)

通过ssh登录VPS,一般是ssh 登录名@ip，输入密码登录即可。由于一般购买VPS之后分配的密码都比较难记，可以通过passwd命令修改root密码。

废话不多说，开始安装Shadowsocks 服务吧！   
> 安装pip环境:  

```
yum install python-setuptools && easy_install pip    
```

![确认截图](http://llyangblog.cn/img/20171020P1.png)

> 然后再安装shadowsock   

```
pip install shadowsocks    
```   

![确认截图](http://llyangblog.cn/img/20171020P1.png)

###2.运行ShadowSocks服务
> 启动命令如下：如果要停止运行，将命令中的start改成stop。

````
sudo ssserver -p 8388 -k password -m aes-256-cfb -d start
````   

> 也可以使用配置文件进行配置，方法创建/etc/shadowsocks.json文件，填入如下内容： 

````   
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
````
> 安装完成之后，启动服务吧，我们就可以使用它了

````
ssserver -c /etc/shadowsocks.json -d start
```` 

###3.加入开机启动，一劳永逸
> 启动之后，每次重启VPS都要重新启动一次ShadowSocks服务，我们可以配置开机启动   
> 创建/etc/rc.d/init.d/autostartss.sh，填入以下内容(注:chkconfig: 2345 80 90一定要添加，否则后面可能会报错):

````
#!/bin/bash
#chkconfig: 2345 80 90
#description:autostartss
ssserver -c /etc/shadowsocks.json -d start
````
> 然后通过chkconfig添加开机启动

````
chmod +x /etc/rc.d/init.d/autostartss.sh
cd /etc/rc.d/init.d/
chkconfig --add autostart.sh
chkconfig autostartss.sh on
````
> 如果没有添加chkconfig: 输入chkconfig --add autostart.sh，可能会出现以下错误，至少我遇到这个问题了。   
![错误](http://llyangblog.cn/img/20171020P1.png)
---
title: vultr搭建ss
comments: true
categories: 科学上网
date: 2019-03-14 09:23:50
tags:
    - vultr
    - ss
---

### 记录下第一次购买vutrl搭建ss

好像作为一个合格的程序员，翻墙好像是一个必备技能，所以也就有`Google`程序员歧视百度程序员这么一说。不管是为了不`被歧视`也好，或者是本着尝试下新知识的目的，我们也来尝试下怎么去搭建个梯子。这里我们选择的方式是[vultr](https://my.vultr.com/),购买很简单，支付宝付好钱后选择服务器（一般是5美元）他自己会自动建好服务器。这个步骤就不多赘述，也可以参看[这篇文章](https://www.vultrblog.com/vultr-ss.html)，我主要记录下VPS建好之后，我们该怎么去链接VPS，搭建ss。

### 连接VPS&配置SS

服务器搭建好之后我们就可以拿到一个IP地址了，我们可以在终端通过ssh连上服务器，只需执行：

`ssh root@155.138.134.216`
<!--more-->
会让你输入密码，当然密码就是在创建服务器的时候已经有了，直接复制粘贴就行。之后就会自动连接了，如果成功连上的话应该是这样的：

```
 ~ $ ssh root@155.138.134.216
root@155.138.134.216's password:
Last failed login: Thu Mar 14 01:12:53 UTC 2019 from 185.42.64.216 on ssh:notty
There were 296 failed login attempts since the last successful login.
Last login: Wed Mar 13 13:24:29 2019 from 183.129.130.102
[root@toronto ~]# 
```
这样我们就可以在远程的vps干点啥事情了，接下来需要安装依赖包：

`curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"`

然后用`python`安装`pip`:

`python get-pip.py`

然后就是安装`shadowsocks`:

`pip install shadowsocks`

安装好ss之后，我们需要配置下ss的配置文件，将我们的ip地址和密码、端口、加密方式等信息填上去。

`vi /etc/shadowsocks.json`

刚打开这个文件是空的，我们改成这样差不多：

```
{
"server": "0.0.0.0",
"server_port": "8000",
"password": "12345678",
"method": "aes-256-cfb"
}
```
很好理解，server是服务器的IP地址，端口也可以随意写，密码是你连ss的密码，根据自己喜好设置，加密方式一般选择这种。这里需要注意下value尽量都要用引号，不然会报错。这个是一个人用，但让想跟小伙伴分享下梯子也可以配置多用户，互不干扰，比如这样：

```
{
"server": "0.0.0.0",
"port_password": {
"8381": "password1",
"8382": "password2",
"8383": "password3",
"8384": "password4"
},
"timeout": 300,
"method": "aes-256-cfb"
}
```
保存，退出。

然后还需要配置下防火墙：

`systemctl stop firewalld.service`

启动SS服务：

`ssserver -c /etc/shadowsocks.json -d start`

启动成功的话，应该可以看到：

```
[root@toronto ~]# ssserver -c /etc/shadowsocks.json -d start
INFO: loading config from /etc/shadowsocks.json
2019-03-14 01:19:39 INFO     loading libcrypto from libcrypto.so.10
started
```

关闭SS服务:

`ssserver -c /etc/shadowsocks.json -d stop`


### 客户端连接SS

[shadowsocks](https://github.com/shadowsocks/ShadowsocksX-NG)可以在github上下载。安装好之后，打开shadowsocks选择服务器配置，按照刚才配置的ss输入服务器的配置信息，点击确定，然后服务器选择刚才添加的服务器。关于PAC自动模式和全局模式、手动模式，根据自己喜好随意切换。好了，搭个梯子好像也就这些，冲浪愉快~




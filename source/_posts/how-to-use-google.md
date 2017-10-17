---
title: 科学上网：用VPS搭建shadowsocks服务器
date: 2015-09-04 21:54:11
tags:
---

学会 科学上网：用 VPS搭建shadowsocks 服务器

目前将shadowsocks部署到25端口似乎还不会被封锁，大家可以尝试。

## 购买VPS服务器

主流的VPS（虚拟主机）服务器提供商有三家：

*   linode
*   digital ocean
*   bandwagon
下面的比上面的便宜。如果只是自用，bandwagon足够。

一般使用paypal绑定一个visa或mastercard信用卡来付款。注意要用国际paypal帐号，国内的是不能用外币付款的。

在bandwagon购买VPS以后会获得一个主机地址和用于ssh登录的root密码。

## 远程登陆VPS

Mac OS X 或Linux下直接在终端中

```
ssh root@your_vps_ip -p your_ssh_port
```
即可。

在windows系统下需要专门的客户端来SSH登录VPS。在[xShell官网](http://www.netsarang.com/download/down_form.html?code=522) 下载xShell。

家庭和学校用户可以免费试用，下载时选择home and school use即可。需要用邮箱注册一下，下载链接会发送到邮箱中。

xShell中新建一个连接，会要求输入目标IP地址和端口，以及root密码，按提示操作即可。

## 安装shadowsocks

打开shell，使用VPS服务商提供的root用户和密码SSH登录VPS。然后执行如下命令：

### Debian/Ubuntu:

```
apt-get install python-pip
pip install shadowsocks
```

### CentOS:

```
yum install python-setuptools &amp;&amp; easy_install pip
pip install shadowsocks
```
shadowsocks就安装好了。

有时Ubuntu会遇到第一个命令安装python-pip时找不到包的情况。pip官方给出了一个安装脚本，可以自动安装pip。先下载脚本，然后执行即可：

```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

## 编写配置文件

shadowsocks启动时的参数，如服务器端口，代理端口，登录密码等，可以通过启动时的命令行参数来设定，也可以通过json格式的配置文件设定。推荐使用配置文件，方便查看和修改。

用vi新建一个配置文件：

```
vi /etc/shadowsocks.json
```
然后输入如下内容：

```
{
"server":"my_server_ip",
"server_port":25,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"mypassword",
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}
```
保存退出。

## 启动shadowsocks

如果已经写好了配置文件，启动shadowsocks服务器的命令如下：

```
ssserver -c /etc/shadowsocks.json
```
后台启动和停止shadowsocks服务器：

```
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```
shadowsocks的日志保存在

```
/var/log/shadowsocks.log
```

## 安装并启动shadowsocks客户端

shadowsocks支持windows、Mac OS X、Linux、Android、iOS等多个平台。不过iOS由于系统对应用后台运行的限制，只能使用客户端内嵌的浏览器科学上网，不能给其他应用代理。

shadowsocks项目Github主页：[https://github.com/shadowsocks/shadowsocks](https://github.com/shadowsocks/shadowsocks)

里面可以找到客户端下载地址。

下载安装客户端以后，只需按服务器的配置填写IP地址、服务器端口、本地端口（如果没有本地端口选项，就是默认的1080）、密码、加密方式等参数，启动就可以了。

客户端支持全局代理和PAC代理两种方式，后者会使用一个脚本来自动检查一个网站是否在需要代理的网站列表中，自动选择直接连接或代理连接。

PAC列表可以在线更新，但是难免有收录不全的情况。这时可以选择关闭shadowsocks代理（实际上是取消对系统代理的配置，shadowsocks客户端仍然保持工作），然后使用支持自定义规则的代理管理插件来实现自动切换代理，比如switchyOmega。

## 使用switchyOmega实现自动切换代理

switchyOmega是chrome浏览器上一个很好用的代理管理插件。它的前身switchySharp更有名。

chrome应用商店本身需要翻墙才能访问，因此需要先在shadowsocks启动代理模式下下载安装，再关闭shadowsocks代理。

安装完毕后，右击switchyOmega图标，选择选项，进入switchOmega配置界面。

### 创建shadowsocks情景模式

新建一个情景模式，比如叫SS，代理协议选择socks5，代理地址为127.0.0.1，端口1080。

现在切换到SS情景模式就可以通过shadowsocks科学上网了。后面获取自动切换规则列表

### 设置自动切换模式

在设置界面选择自动切换模式，在“切换规则”中勾选“规则列表规则”，对应的情景模式选择刚刚新建的SS。

然后在下面的规则列表地址中填写

[https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt](https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt)

规则列表格式选择AutoProxy。

然后点击立即更新情景模式， 更新完成后会有提示。

点击左侧的“应用选项”。然后单击switchyOmega图标，选择自动切换，就可以在访问“不存在的网站”时自动切换到shadowsocks代理了。

### 添加自定义规则

如果遇到某个国外网站无法直接连接或速度太慢时，可以单击switchyOmega图标，选择“添加条件”，情景模式选择SS，就可以了。

这时打开switchyOmega选项，在自动切换模式的切换规则中就可以看到刚刚添加的规则。可以在这里管理自定义的规则。

### 导入和导出switchyOmega设置

如果换了一台电脑，重新设置一遍switchyOmega就太麻烦了。可以在设置好的switchyOmega中导出设置文件，在另一个chrome浏览器中导入，就可以直接复制原来的设置了。

在switchyOmega选项的左侧点击“导入/导出”，点击“生成备份文件”即可生成switchyOmega设置备份。点击“从备份文件恢复”可以导入备份文件。
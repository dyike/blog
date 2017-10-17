---
title: ubuntu下配置nginx，mysql，php环境
tags:
  - PHP
  - 工具
  - 虚拟机
date: 2015-09-16 20:56:59
---


## 概述 ##

今天在虚拟机上基于Ubuntu安装配置了服务器运行环境，web服务用 nginx，数据库存储在 mysql,动态脚本语言是 php。


#### 安装nginx ####
##### 下载并添加nginx_signing.key #####
```
wget http://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
```
##### 添加官方源 #####
```
sudo vim /etc/apt/sources.list
```
在末尾加上：

```
deb http://nginx.org/packages/ubuntu/ trusty nginx
deb-src http://nginx.org/packages/ubuntu/ trusty nginx
```
然后更新：

```
sudo apt-get update
```
##### 安装nginx #####
```
sudo apt-get install nginx
```



#### 安装mysql ####

##### 下载和安装MySQL APT Repository #####
```
wget http://dev.mysql.com/get/mysql-apt-config_0.5.3-1_all.deb
sudo dpkg -i mysql-apt-config_0.5.3-1_all.deb
```
##### 确认版本，或者更改版本：#####
```
sudo dpkg-reconfigure mysql-apt-config
```
##### 更新源并安装mysql-server #####
```
sudo apt-get update
sudo apt-get install mysql-server
```

#### 安装PHP ####
##### 下载Php并解压 ####

```
wget http://cn2.php.net/distributions/php-7.0.6.tar.bz2
tar jxf php-7.0.6.tar.bz2
```

##### 安装所需依赖 #####

```
sudo apt-get install libxml2-dev build-essential openssl libssl-dev make curl libcurl4-gnutls-dev libjpeg-dev libpng-dev libmcrypt-dev libreadline6 libreadline6-dev
```
##### configure #####

```
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-mbstring --with-mcrypt --enable-ftp --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --disable-fileinfo --enable-maintainer-zts
```

或者是这样的：(下面是我常用的)
```
./configure --prefix=/usr/local/php7 \
--with-config-file-path=/usr/local/php7/etc \
--with-config-file-scan-dir=/usr/local/php7/etc/php.d \
--with-mcrypt=/usr/include \
--enable-mysqlnd \
--with-mysqli \
--with-pdo-mysql \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--with-gd \
--with-iconv \
--with-zlib \
--enable-xml \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--with-openssl \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \
--with-curl \
--with-jpeg-dir \
--with-freetype-dir \
--enable-opcache  \
--with-readline
```
```
sudo make && make install 
```

##### 加入系统变量 #####
```
sudo echo "PATH=$PATH:/usr/local/php7/bin">> /etc/profile
sudo echo "export PATH">> /etc/profile
source /etc/profile
```

##### 一些配置文件的复制 #####

* 编译后，cd到源码跟目录，然后

```
sudo cp php.ini-production /usr/local/php7/lib/php.ini
```
* cd /usr/local/php7/etc ,然后

```
sudo cp php-fpm.conf.default php-fpm.conf
```
* cd /usr/local/php7/etc/php-fpm.d，然后

```
sudo cp www.conf.default www.conf
```

##### 设置php-fpm配置文件 #####
* php-fpm.conf的pid

```
sudo vim /usr/local/php7/etc/php-fpm.conf
```


* 启动php-fpm服务

```
/usr/local/php7/sbin/php-fpm
```

* 新建www用户组：

```
groupadd www
useradd -g www www
```

* 再次启动

```
/usr/local/php7/sbin/php-fpm
```

* 开机启动php-fpm
开机启动的配置文件是：/etc/rc.local ，加入 /usr/local/php7/sbin/php-fpm 即可

```
 vi /etc/rc.local
```
```
#!/bin/sh

# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.
     
touch /var/lock/subsys/local
/usr/local/apache/bin/apachectl start
/usr/local/bin/redis-server /etc/redis.conf
/usr/local/php7/sbin/php-fpm
```      

 
 
#### 配置nginx略了 ####




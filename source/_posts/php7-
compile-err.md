---
title: php7编译出现的问题
tags:
  - PHP
date: 2016-01-02 18:15:13
---

在编译安装php7时日志中有如下错误记录：
virtual memory exhausted: Cannot allocate memory
make: *** [ext/fileinfo/libmagic/apprentice.lo] Error 1

开始时的解决方法是先将很多的进程关闭，比如httpd、ftpd、sendmail等等，释放出了一部分内存后，再进行编译，仍然得到同样的编译错误。后来百度谷歌了很久才找到解决问题的方法，而且是在php.net上找到的解决方法，原文链接是：https:  //bugs.php.net/bug.php?id=48809

根据这个文章的方法将php安装配置文件中加了引号中的配置（不包括引号）“- -disable-fileinfo”后终于编译通过。
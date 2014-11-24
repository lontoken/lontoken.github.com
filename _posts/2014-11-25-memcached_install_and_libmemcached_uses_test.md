---
layout: post  
title: memcached安装和libmemcached的使用test
tags: [memcached,安装,libmemcached]  
---

memcached安装和libmemcached的使用test
====

#环境和版本#
>    操作系统：Ubuntu14.04 32bit  
>    libevent版本: 2.0.21  
>    memdatach版本: v1.4.21  


#libevent安装#

{% highlight sh %}
#wget http://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
#tar -xvzf libevent-2.0.21-stable.tar.gz
#cd libevent-2.0.21-stable
#./configure -prefix=/usr
#make
#make install
{% highlight %}

<!--more-->

查看是否安装成功:  

{% highlight sh %}
#ls /usr/lib/ | grep  libevent
{% highlight %}


#memcached安装#

{% highlight sh %}  
#wget wget http://www.memcached.org/files/memcached-1.4.21.tar.gz
#tar -xvzf memcached-1.4.21.tar.gz
#cd memcached-1.4.21
#./configure -with-libevent=/usr
#make
#make install
{% highlight %}
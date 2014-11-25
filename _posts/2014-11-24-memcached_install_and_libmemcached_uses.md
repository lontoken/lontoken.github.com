---
layout: post  
title: memcached安装和libmemcached的使用  
tags: [memcached,安装,libmemcached]
published: true
---

memcached安装和libmemcached的使用  
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

查看是否安装成功: 

{% highlight sh %}
#ll /usr/local/bin
{% highlight %}


#memcached启动#

{% highlight sh %}
#/usr/local/bin/memcached -d -u root -m 512 127.0.0.1 -p 11211
{% highlight %}

查看侦听端口和进程信息：  

{% highlight sh %}
#netstat -a |grep 11211
#ps -ef | grep memcached
{% highlight %}


#测试memcached#
连接memcached最简单的方法是通过telnet。  

{% highlight sh %}
#telnet 127.0.0.1 11211
{% highlight %}

查看memcached的状态(telnet下执行):    

{% highlight sh %}
stats
{% highlight %}

键值简单的设置、查看和删除(telnet下执行):  

{% highlight sh %}
set user_id 0 0 5
12345
get user_id
delete user_id
get user_id
{% highlight %}

PS:退出telnet，可以键入alt+] q 


#libmemcached安装#

{% highlight sh %}  
#wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
#tar -xvzf libmemcached-1.0.18.tar.gz
#cd libmemcached-1.0.18
#./configure
#make
#make install
{% highlight %}

查看libmemcached是否安装成功:  

{% highlight sh %}
#ls /usr/local/lib | grep libmemcached
{% highlight %}


#使用C++通过libmemcached连接memcached# 
C++源文件 libmemcachedtest.cpp  

{% highlight c++ linenos %}
#include <iostream>
#include <string>
#include <libmemcached/memcached.h>

using namespace std;

int main(int argc, char *argv[])
{
    //connect server
    cout << "test start" << endl;
    memcached_st *memc;
    memcached_return rc;
    memcached_server_st *server;
    uint32_t  flags;

    memc = memcached_create(NULL);
    cout << "append start" << endl;
    server = memcached_server_list_append(NULL, "localhost", 11211, &rc);
    if(rc != MEMCACHED_SUCCESS){
        cout << "memcached_server_list_append failed. rc=" << rc << endl;
        return -1;
    }

    rc = memcached_server_push(memc, server);
    if(rc != MEMCACHED_SUCCESS){
        cout << "memcached_server_push failed. rc=" << rc << endl;
        memcached_server_free(server);
        return -2;
    };

    memcached_server_list_free(server);

    string key = "key";
    string value = "value";
    size_t value_length = value.length();
    size_t key_length = key.length();

    //Save data
    cout << "save data" << endl;
    rc = memcached_set(memc, key.c_str(), key_length, value.c_str(), value_length, 0, flags);
    if(rc == MEMCACHED_SUCCESS){
        cout << "save data sucessful, key=" << key << ",value=" << value <<endl;
    }else{
        cout << "save data faild, rc=" << rc <<endl;
    }

    //get data
    cout << "get data" << endl;
    char* result = memcached_get(memc, key.c_str(), key_length, &value_length, &flags, &rc);
    if(rc == MEMCACHED_SUCCESS){
        cout << "get value sucessful, result=" << result <<endl;
    }else{
        cout << "get value faild, rc=" << rc <<endl;
    }

    //delete data
    cout << "delete data" << endl;
    rc = memcached_delete(memc, key.c_str(), key_length, 0);
    if(rc == MEMCACHED_SUCCESS){
        cout << "delete key sucessful. key=" << key << endl;
    }else{
        cout << "delete key faild, rc=" << rc <<endl;
    }

    //free
    memcached_free(memc);
    cout << "test end." << endl;
    return 0;
}
{% highlight %}

编译前需要设置LD_LIBRARY_PATH环境变更，以使libmemcached能被找到。 

{% highlight sh %}
$export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/
{% highlight %}
 
编译并执行：

{% highlight sh %} 
$g++ -std=c++11 -o libmemcachedtest libmemcachedtest.cpp -lmemcached
$./libmemcachedtest
{% highlight %} 
 
如果一切顺利，输出如下： 

{% highlight sh %}
test start
append start
save data
save data sucessful, key=key,value=value
get data
get value sucessful, result=value
delete data
delete key sucessful. key=key
test end.
{% highlight %}

-----------------
本文结束，若有错误和疑问，欢迎交流(邮件：lontoken@gmail.com)。  

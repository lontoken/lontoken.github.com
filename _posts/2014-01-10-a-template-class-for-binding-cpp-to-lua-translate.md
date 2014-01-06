---
layout: post
title: A template class for binding C++ to Lua(翻译)
tags: [lua,翻译,C++]
published: false
---
#A template class for binding C++ to Lua
英文原文：[A template class for binding C++ to Lua](http://www.lua.org/notes/ltn005.html)

##摘要
此文介绍了一种将C++类绑定到Lua的方法。Lua没有直接提供此方法，而是通过底层的C接口和扩展机制使其成为可能。我所描述的方法使用了Lua的C接口、C++模板和Lua的提高扩展机制，构建了一个小巧、简单且高效的提供类注册服务的静态模板类。这个方法对你的类只有一个小小的要求，即只有签名为 int(T::*)(lua_State*) 的成员函数能被注册。但是，正如我将要展示的，这个限制也能被克服。The end result is a clean interface to register classes, and familiar Lua table semantics of classes in Lua。此处描述的解决方案依赖于一个我命名为[Luna](http://lua-users.org/files/wiki_insecure/users/lpalozzi/luna.tar.gz)的模板类。

##问题
Lua的接口没被设计来注册C++类到Lua中，只提供了注册签名为 int()(lua_State*) 的C函数。实际上，这是Lua支持注册的唯一C数据类型。为了注册其它类型，你需要使用Lua提供的扩展机制，如tag methods、closures等。为了创建允许我们注册C++类到Lua的方案，必须使用这些扩展机制。

##方案
此方案主要有4个元素：类注册、对象实例化、成员函数调用和垃圾回收。

类注册是通过以类的名字注册一个表构造函数(a table constructor function with the name of the class)。表构造函数一个在lua_State中注册一个表的静态模板函数。

注释：静态类成员函数是和C函数兼容的，如果它们的签名相同，则我们可以在Lua中注册它们。下面的代码片段是一个模板类的成员函数，T 是待注册的类。

{% highlight lua linenos %}
static void Register(lua_State* L) {
    lua_pushcfunction(L, &Luna<T>::constructor);
    lua_setglobal(L, T::className);

    if (otag == 0) {
        otag = lua_newtag(L);
        lua_pushcfunction(L, &Luna<T>::gc_obj);
        lua_settagmethod(L, otag, "gc"); /* tm to release objects */
    }
}
{% endhighlight %}

对象实例化是通过
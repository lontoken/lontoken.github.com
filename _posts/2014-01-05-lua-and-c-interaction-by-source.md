---
layout: post
title: lua和C交互--源码分析
tags: [lua,C,lua和C,lua源码]
published: false
---

#lua和C交互--源码分析
最近在读Lua的源码，之前在Lua中调用C导出的函数和导出的userdata，觉得很方便，没去深究，不知道自己的代码在栈层面是个如何情形。现在借此机会，将整个流程弄明白，以供分享。

<!--more-->


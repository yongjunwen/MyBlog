---
title:   iOS-ARC和垃圾回收机制的区别
tags:  [iOS-WIKI]
categories:  [iOS]
date:  2018-06-11 16:17:22
---


在`java`中有个垃圾回收器，什么是垃圾回收器，就是在程序运行的期间，垃圾回收器会不断的去扫描堆中的对象是否无人使用。如果无人使用就会回收了。

`ARC`在编译的时候就会在合适的地方插入`retain,release,autorelease`,在运行的时候没有东西去进行实时检查，它插入的代码可以在对象没有人用的时候，引用计数器就会变为0.一旦变为0就会被销毁了。

如果在java中有一段代码是`Person * p=[Person new]; p=nil`，这个对象什么时候被回收？不一定，因为这要看垃圾回收器有没有扫描到这里。


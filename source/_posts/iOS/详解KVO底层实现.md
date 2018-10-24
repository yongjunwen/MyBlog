---
title:  详解KVO底层实现
tags:  [iOS-WIKI]
categories:  [iOS]
date:  2018-06-11 16:17:22
---



### 一、KVO (Key-Value Observing)
KVO 是 Objective-C 对观察者模式（Observer Pattern）的实现。也是 Cocoa Binding 的基础。当被观察对象的某个属性发生更改时，观察者对象会获得通知。

有意思的是，你不需要给被观察的对象添加任何额外代码，就能使用 KVO 。这是怎么做到的？

### 二、 KVO内部实现原理
KVO是基于runtime机制实现的
当某个类的属性对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter 方法。派生类在被重写的setter方法内实现真正的通知机制
如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
每个类对象中都有一个isa指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将isa指针指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的setter方法
键值观察通知依赖于NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey:一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey:会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。
补充：KVO的这套实现机制中苹果还偷偷重写了class方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类

 
### 三、补充： 如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？（看链接）

[如何自己动手实现 KVO](https://tech.glowing.com/cn/implement-kvo/)
### 四、附注: KVC底层实现原理(如下)

KVC运用了一个isa-swizzling技术. isa-swizzling就是类型混合指针机制, 将2个对象的isa指针互相调换, 就是俗称的黑魔法.
KVC主要通过isa-swizzling, 来实现其内部查找定位的. 默认的实现方法�由NSOject提供isa指针, 如其名称所指,(就是is a kind of的意思), 指向分发表对象的类. 该分发表实际上包含了指向实现类中的方法的指针, 和其它数据。

具体主要分为三大步  
第一步：寻找该属性有没有setsetter方法？有，就直接赋值  
第二步：寻找有没有该属性带下划线的成员属性？有，就直接赋值  
第三步：寻找有没有该属性的成员属性？有，就直接赋值  
或者这么说  
1、首先搜索setKey:方法.(key指成员变量名, 首字母大写)  
2、上面的setter方法没找到, 如果类方法accessInstanceVariablesDirectly返回YES. 那么按 _key, _isKey，key, iskey的顺序搜索成员名.(NSKeyValueCodingCatogery中实现的类方法, 默认实现为返回YES)  
3、如果没有找到成员变量, 调用setValue:forUnderfinedKey:  
比如说如下的一行KVC的代码：

举个🌰e.g:

``` objc
[object setValue:@"13123" forKey:@"uuid"];
 
就会被编译器处理成:
// 首先找到对应sel
SEL sel = sel_get_ uuid("setValue:forKey:");
// 根据object->isa找到sel对应的IMP实现指针
IMP method = objc_msg_lookup (object->isa,sel);
// 调用指针完成KVC赋值
method(object, sel, @"13123", @"uuid");
```


### 如何为一个实例动态替换方法
[资料](http://ios.jobbole.com/93080/)  
根据kvo的实现原理：派生个实例方法所在类的子类，创建hook函数，然后修改isa指针`object_setClass(instance, newClass);`，实现替换的方法实现。

[详解KVO底层实现](https://www.cnblogs.com/oc-bowen/p/5884768.html)  
[代码实现仓库地址](https://coding.net/u/dream001/p/CustomKVO/git/tree/master)

---
layout: post
title:  "Python标准库之pickle"
date:   2018-04-14 10:20:00 +0800
categories: translator
tags: Python
---

# pickle — Python对象序列化 #

**pickle**模块实现了python对象结构的序列化和反序列化的二进制协议。"Pickling"是一个通过python对象结构转化为二进制流的过程，"unpickling"是一个相反的操作，通过字节流转换为对象结构。Pickling(和unpickling)也被称为“serialization”, “marshalling,” [1] or “flattening”。但是，为了避免混乱，这里
统一使用“pickling"和”unpickling"作为术语。

## 与python其他模块联系 ##
Python有一个更简单的序列化模块，叫**marshal**，但通常pickle应该总是优先方法来序列化python对象。**marshal**的存在主要是为了支持python的**.pyc**文件。
**pickle**模块与**marshal**不同之处在于：
* pickle模块会追踪已经被序列化的对象，所以后面的对相同对象的引用不会再次实例化。**marshal**不会这样做。  
这对循环对象和对象共享都有影响。循环对象是那些包含自身引用的对象。这些在marshal里面不会被操纵，但实际上，尝试对循环对象进行marshal会使python编译器奔溃。对象共享发生在有多个不同地方的引用指向同一个对象，并正在序列化。pickle只存储这种对象一次，并且确保多有其他引用指向母版。共享对象还是共享的，这对可变对象来说很重要。
* marshal不能实例化用户定义的类和它们的实例。pickle可以透明地保存和还原类的实例，但是，类定义必须是可导入的并且与对象存储的时候在同一模块里。
* marshal序列化格式不能被保证在不同python版本中的可移植性。因此它主要的工作是支持**.pyc**文件。python的实现者保留了在需求出现时以非向后兼容的方式更改序列化格式的权利。pickle序列化格式能被保证在python发型本中向后兼容。

## 与json比较 ##
pickle协议与json之间有一些基本区别：
* JSON是一个文本序列化格式（它输出unicode文本，虽然大多数时间它被编码为utf-8）,但是pickle是一个二进制序列化格式。
* JSON是可供人阅读的，但是pickle不是。
* JSON是互操作性的，能在python生态系统以为使用。但是pickle智能在python。
* JSON，默认下，只能代表Python内置类型的子集，没有定制类型。pickle可以表示大量的python类型。（其中许多是自动的，通过巧妙地使用Python的内省工具;复杂的情况可以通过实现特定的对象api来解决

# 数据流格式 #
pickle使用的数据格式是python特定的。这样有一个好处，不会被外部的标准限制。比如JSON或者XDR。但是这也意味着非python项目不能重建pickle的python对象。  
默认下，pickle数据格式使用相对紧凑的二进制表示。如果你需要最佳的尺寸特性，你可以有效地压缩pickle的数据。  
pickletools模块包含用于分析pickle生成的数据流的工具pickletools源代码对pickle协议使用的操作码有广泛的注释。  
当前有5种不同协议用来pickling.	使用的协议越高，就越需要最近的版本的python读取所生成的pickle。
* 协议版本0是原始的可供人阅读的协议，并向后兼容，在早期python版本里面。
* 协议版本1是旧的二进制协议与早期python版本兼容
* 协议版本2在python2.3中引入。它提供了更高效新形类pickling。参考PEP307,了解协议2带来的改进信息。
* 协议版本3在python3.0中加入。它有对比特对象明确的支持且不能被python2.x反序列化。这是一个默认的协议，当要与其他python3版本兼容，这是推荐的协议。
* 协议版本4在python3.4中加入。它增加了对大对象的支持，能序列化更多种类的对象和一些数据格式的优化。参考PEP3154，了解协议4带来的改进。

> 注意：  序列化是必持久化更基础的概念。虽然pickle读写文件对象，但是它不处理命名持久对象的问题，也不处理对持久对象并发访问的问题。pickle模块把一个复杂对象转为字节流。它也可以转换字节流为同样内部结构的对象。可能对这些字节流最明显的事就是把他们写到文件里，但也可能把他们发送到网络或存在数据库里面。shelve模块提供了简单的接口去序列化和反序列化在dbm样式数据库文件上的对象。

# 模块接口 #
为了序列化一个对象层次，你只需要调用dumps()函数。相似的，反序列化一个数据流，你需要调用loads()函数。但是，如果你需要更多序列化和反序列化的控制，你可以分别创建一个Pickler或者Unpickler对象
pickler模块提供了下面的常量：  
pickle.**HIGHEST_PROTOCOL**  
一个整形，可用的最高协议版本。这个值可以作为协议值传递给dump()和dumps()以及Pickler构造器。  
pickle.**DEFAULT_PROTOCOL**  
一个整形，pickling的默认协议版本。可能小于HIGHEST_PROTOCOL。当前，默认协议是3，为python3设计的最新协议。
pickele模块提供了下列的函数来使序列化过程更简便。
......（这里不想翻译咯）

# 什么可以被序列化与反序列化 #
下列的类型可以被序列化：
* **None**, **True** 和 **False**
* 整型，浮点型数字，复杂数字
* 字符，比特，比特数组
* 元组，列表，集合和字典，只包含可序列化对象
* 定义在顶级模块中的函数（使用def,而不是lambda）
* 定义在顶级模块的自建函数
* 定在在顶级模块的类
* __dict__或者调用__getstate__()的结果可以学历恶化的对象的实例

 



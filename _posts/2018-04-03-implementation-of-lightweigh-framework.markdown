---
layout: post
title:  "轻量级框架Milky设计"
date:   2018-04-03 18:20:09 +0800
categories: Java
---

## **注入方式** ##
### **构造器注入** ###
在构造器注入上提供了下列标签 **index**, **value**, **type**, **ref** 四种属性。其中**index** 属性指定构造器中参数的顺序，**value** 属性指定注入的值，**type** 属性指定注入的类型，**ref** 属性指定注入的引用类型。  
注入基本类型时，可以使用如下方法：  
```
	<constructor-arg index='0' value='1' type='int' \>
```

注入引用类型时，可以使用如下方法：  
```
	<constructor-arg index='0' ref='newBean'\>
```  
由于引用类型被容器管理，并有唯一Id映射，不需要指定类型。




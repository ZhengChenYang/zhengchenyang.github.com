---
layout: post
title:  "轻量级框架Milky设计"
date:   2018-04-03 18:20:09 +0800
categories: 小项目
tags: Java 轻量级框架
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


## **set方法注入** ##
使用set方法的注入需要有两个保证。第一，别注入的对象首先具有该属性的set方法。方法名的定义必须满足以下格式，"set"+属性名的格式，并以遵循驼峰命名法。在xml配置上，提供了**\<property\>**标签作为set方法注入的配置。**\<property\>**标签有以下几个属性：
1. **name**  
name指定要需要注入的属性
2. **value**  
value指定要注入的属性值
3. **ref**  
ref指定要注入的引用对象


## **XML标签解析器** ##
向BeanDefinitionParserDelegate中注册XML标签解析器，可以扩展新的标签。BeanDefinitionParserDelegate中包含了默认标签，默认便签提供了对应的默认解析器。默认标签包含以下：
1. bean
2. constructor-arg
3. array
4. list
5. set
6. map
7. property  

标签解析器需要实现XmlTagParser接口中的parse()方法。   

**constructor-arg** 标签解析步骤（可以加一个流程图哦！）：
1. 迭代中找到constructor-arg标签，进行处理
2. 获取index属性和type属性
3. 解析属性值
4. 如果标签包含ref属性，则直接填充ref字段，如果标签包含value属性，则直接填充value。如果有子标签，进一步进行解析。否则到6步完成解析
5. 解析子标签，根据便签名，对集合元素进行解析。解析得到的值放入到ValueHolder对象中，如果该对象继承了TypeConverter接口，在容器实例化bean的时候，会调用TypeConverter接口中的convert()方法，进行转换。
6. 完成解析




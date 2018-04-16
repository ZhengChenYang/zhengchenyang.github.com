---
layout: post
title:  "Python标准库之pickle"
date:   2018-04-14 10:20:00 +0800
categories: translator
tags: Python
---

# pickle — Python对象序列化 #
**pickle**模块实现了python对象结构的序列化和反序列化的二进制协议。"Pickling"是一个通过python对象结构转化为二进制流的过程，"unpickling"是一个相反的操作，通过字节流转换为对象结构。Pickling(和unpickling)也被称为“serialization”, “marshalling,” 或者 “flattening”。但是，为了避免混乱，这里
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
* 定义在模块顶层中的函数（使用def,而不是lambda）
* 定义在模块顶层的自建函数
* 定在在模块顶层的类
* \_\_dict\_\_或者调用\_\_getstate\_\_()的结果可以序列化的类的实例
  
尝试序列化不可序列化的对象会抛出PicklingError异常。当这个发生的，可能已经将一个未指明的字节数写入到底层的文件中。尝试序列化一个高度递归的数据结构到最大递归深度，这种情况下一个 RecursionError会被抛出在。你可以用sys.setrecursionlimit()小心提高这个限制。  
注意，函数（包括内置和用户定义）是由完全限定名字引用，而不是值。这就意味着只有函数名能被序列化，以及所在模块的名字。函数的代码以及里面的属性都不会被序列化。定义的模块必须在反序列化环境中导入，模块必须包含命名对象，否则会抛出异常。  
相似的，类通过命名引用被序列化，相同的限制会在反序列化环境中采用。注意，类的代码和数据都不会被序列化，所以在下面的例子中，class中的属性attr不会被恢复在反序列化环境中。  
```python
class Foo:
    attr = 'A class attribute'

picklestring = pickle.dumps(Foo)
```

这些限制就是为什么，要序列化的函数和类必须被定义在模块的顶层。
相似的，当类的实例被序列化，他们的类代码和数据不会跟着他们。只有实例数据会被序列化。这样做是有原因的，你可以修复bug在一个类中，或者增加新的方法给类，以及加载早期版本的类创建的对象。如果你计划有一个能看到多个版本类的长寿对象，这值得你去把版本号作为属性放到对象中，所以适当的转换可以被类的\_\_setstate\_\_方法调用。

# 序列化类实例 #
在这个单元，我们描述一种普遍的机制可供你定义，定制以及控制类实例如何被序列化和反序列化。  
在大多数的情况下，不需要附加的代码使实例可序列化。默认下，pickle会通过内省检索类以及实例的属性。当一个类实例被反序列化，它的\_\_init\_\_()方法通常不会被调用。默认的行为是先创建一个未被初始化的实例以及恢复存储的属性。下面的代码展示了这种行为的实现：  
```python

	def save(obj):
		return (obj.__class__, obj.__dict__)
	
	def load(cls, attribute):
		obj = cls.__new__(cls)
		obj.__dict__update(attributes)
		return obj
```  

类可以选择默认的行为通过提供一种或几种特殊的方法  
1. object.**\_\_getnewargs_ex\_\_()**  
在协议2以及更新的版本中，实现__getnewargs_ex_()方法的类可以指令在反序列化时传递给\_\_new\_\_()的值。  
你应该实现这个方法，如果\_\_new\_\_()方法要求关键字参数。否则，为了兼容，推荐实现\_\_getnewages\_\_().
2.   object.**\_\_getnewargs\_\_()**  
这个方法和\_\_getnewargs\_ex\_\_()有相似的目的，单只支持位置参数。它必须返回一个参数args的元组，这个参数会被传递给\_\_new\_\_()方法在反序列化时候。\_\_getnewargs\_\_()不会被调用，如果\_\_getnewargs\_ex\_\_()被定义。  
在版本3.6中发生翻遍，在python3.6之前，\_\_getnewargs\_\_()会被调用来代替\_\_getnewargs\_ex\_\_()在协议2和协议3中。
3. object.**\_\_getstate\_\_()**
类可以进一步影响实例如何被序列化。如果类定义了方法\_\_getstate\_\_(),它会被调用并返回对象作为序列化的内容，作为实例字典的代理。如果\_\_getstate\_\_()方法缺少，实例的\_\_dict\_\_会像往常一样被实例化。
4. object.**\_\_setstate\_\_(state)**  
在反序列时，如果类定义了\_\_state\_\_()，它会在反序列化的时候调用。在那种情况下，不需要state对象是一个字典。否则，序列化状态必须是一个字典，它的条目会被赋值给新的实例字典。
> 注意：如果\_\_getstate\_\_()返回一个false，\_\_setstate\_\_方法不会在反序列的时候调用。   

正如我们所看到的，序列化不会直接调用上述所说的方法。实际上，这些方法复制协议的一部分，它实现了\_\_reduce\_\_()特殊方法。复制协议提供了统一的接口来检索数据给序列化和复制对象。  
尽管很强大，实现\_\_reduce\_\_()在你的类里是易于出错的。为了这个原因，类设计者应该尽可能使用更高层的接口（\_\_getstate\_\_()等）。尽管，我们也会展示一些情况下，\_\_reduce\_\_()是唯一的选择或者会有更高效的序列化或者两者都会。
5. object.**\_\_reduce\_\_()**  
这个接口当前是下面这样定义的。\_\_reduce\_\_()方法不接受任何参数，必须返回字符串或者更好是元组（返回的值通常被称为reduce值）。
如果一个字符串返回，字符串应该被解释为全局变量的名称。它应该是对象的本地变量相对于它的模块。pickle模块搜索模块的命名空间决定对象的模块。这种行为通常对单例有效。  

当一个元组被返回，它必须有2到5个条目长。可选项可以省略，也可以是None作为值。每个条目根据语义排顺序：
* 一个可调用的对象，它将被调用来创建对象的初始版本。
* 可调用对象的一组参数。如果callable不接受任何参数，则必须给出空元组。
* 可选的。对象的状态，会被传递给对象的\_\_setstate\_\_()方法前面所提到。如果对象没有这种方法，这个值必须是字典，并且会被加到对象的\_\_dict\_\_值上。
* 可选的。一个迭代器（不是一个序列）产生联系的条目。这些条目会被追加到对象上，要么使用**obj.append(item)**，或者，用批处理，使用**obj.extend(list_of_items)**。这个主要用于list子类，但是可以被其它有append()和extend()方法的类使用，只要有适当的签名。（是否append()和extend()被使用取决于，哪种pickle协议版本被使用，以及追加的条目数，这两个都是要的）
* 可选的。一个迭代器（不是序列）产生连续的键值对。这些项目会被存储在对象里，使用obj[key]=value.这个主要用于dictionary子类，也可能被其它实现了\_\_setitem\_\_()的类使用。

# 外部对象持久化 #
为了对象持久化的好处，pickle模块支持pickle数据流之外的对象引用概念。这种对象呗persistent ID应用，它应该是一串数字字母（协议1），或者就是一个随机对象（对于更新的协议）。  
persistent ID方案不是被pickle模块定义。它会委托这个处理给用户自动以的方法在pickler和unpickler上，分别对应persistent_id()和persistent_load()。  
为了序列化有外部persistent Id的对象，pickler必须有一个定制的persistent_id()方法，它以一个对象作为参数，返回None或者那个对象的persistent Id。当None返回的时候，pickler仅仅序列化对象像平常一样。当一个persistent id字符串返回，pickler会序列化那个对象，以及有一个书签，这样unpickler会重新识别它为persistent id。  
为了反序列化外部对象，unpickler必须有一个定制的persisrent_load()方法，以一个persistent Id作为参数，返回一个引用对象。  
这有一个全面的例子展示persisrent id如何通过引用被用来序列化外部对象。
```python

# Simple example presenting how persistent ID can be used to pickle
# external objects by reference.

import pickle
import sqlite3
from collections import namedtuple

# Simple class representing a record in our database.
MemoRecord = namedtuple("MemoRecord", "key, task")

class DBPickler(pickle.Pickler):

    def persistent_id(self, obj):
        # Instead of pickling MemoRecord as a regular class instance, we emit a
        # persistent ID.
        if isinstance(obj, MemoRecord):
            # Here, our persistent ID is simply a tuple, containing a tag and a
            # key, which refers to a specific record in the database.
            return ("MemoRecord", obj.key)
        else:
            # If obj does not have a persistent ID, return None. This means obj
            # needs to be pickled as usual.
            return None


class DBUnpickler(pickle.Unpickler):

    def __init__(self, file, connection):
        super().__init__(file)
        self.connection = connection

    def persistent_load(self, pid):
        # This method is invoked whenever a persistent ID is encountered.
        # Here, pid is the tuple returned by DBPickler.
        cursor = self.connection.cursor()
        type_tag, key_id = pid
        if type_tag == "MemoRecord":
            # Fetch the referenced record from the database and return it.
            cursor.execute("SELECT * FROM memos WHERE key=?", (str(key_id),))
            key, task = cursor.fetchone()
            return MemoRecord(key, task)
        else:
            # Always raises an error if you cannot return the correct object.
            # Otherwise, the unpickler will think None is the object referenced
            # by the persistent ID.
            raise pickle.UnpicklingError("unsupported persistent object")


def main():
    import io
    import pprint

    # Initialize and populate our database.
    conn = sqlite3.connect(":memory:")
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE memos(key INTEGER PRIMARY KEY, task TEXT)")
    tasks = (
        'give food to fish',
        'prepare group meeting',
        'fight with a zebra',
        )
    for task in tasks:
        cursor.execute("INSERT INTO memos VALUES(NULL, ?)", (task,))

    # Fetch the records to be pickled.
    cursor.execute("SELECT * FROM memos")
    memos = [MemoRecord(key, task) for key, task in cursor]
    # Save the records using our custom DBPickler.
    file = io.BytesIO()
    DBPickler(file).dump(memos)

    print("Pickled records:")
    pprint.pprint(memos)

    # Update a record, just for good measure.
    cursor.execute("UPDATE memos SET task='learn italian' WHERE key=1")

    # Load the records from the pickle data stream.
    file.seek(0)
    memos = DBUnpickler(file, conn).load()

    print("Unpickled records:")
    pprint.pprint(memos)


if __name__ == '__main__':
    main()

```

# 调度表 #
如果想要定制某些类的的pickle,而不影响其他依赖于pickle的代码，那么可以创建一个带有私人调度表的pickler。  
受copyreg模块管理的全局调度表可以作为copyreg.dispatch_table.因而，你可以选择修改后的copy.dispatch_table副本作为私人调度表。  
For example
```python
	f = io.BytesIO()
	p = pickle.Pickler(f)
	p.dispatch_table = copyreg.dispatch_table.copy()
	p.dispatch_table[SomeClass] = reduce_SomeClass
```

创建一个pickler的实例，拥有私人调度表，它可专门操作SomeClass。作为一种选择的代码：
```python
	class MyPickler(pickle.Pickler):
	    dispatch_table = copyreg.dispatch_table.copy()
	    dispatch_table[SomeClass] = reduce_SomeClass
	f = io.BytesIO()
	p = MyPickler(f)
```
上面代码做了相同的事情，不过所有的MyPickler实例都会默认分享相同的调度表。使用copyreg模块相同的代码：  
```python
	copyreg.pickle(SomeClass, reduce_SomeClass)
	f = io.BytesIO()
	p = pickle.Pickler(f)
```

# 处理状态对象 #
这里有一个例子展示如何修改序列化行为。TextReader类打开了一个文本文件，返回了行数和每次调用readline()返回每行内容。如果TextReader实例被序列化，所有的属性除了文件对象成员对会被保存。当实例被反序列化，文件重新被打开，继续读取上一次到达的位置。\_\_getstate\_\_()和\_\_setstate\_\_()方法被用来实现这样的行为。
```python
class TextReader:
    """Print and number lines in a text file."""

    def __init__(self, filename):
        self.filename = filename
        self.file = open(filename)
        self.lineno = 0

    def readline(self):
        self.lineno += 1
        line = self.file.readline()
        if not line:
            return None
        if line.endswith('\n'):
            line = line[:-1]
        return "%i: %s" % (self.lineno, line)

    def __getstate__(self):
        # Copy the object's state from self.__dict__ which contains
        # all our instance attributes. Always use the dict.copy()
        # method to avoid modifying the original state.
        state = self.__dict__.copy()
        # Remove the unpicklable entries.
        del state['file']
        return state

    def __setstate__(self, state):
        # Restore instance attributes (i.e., filename and lineno).
        self.__dict__.update(state)
        # Restore the previously opened file's state. To do so, we need to
        # reopen it and read from it until the line count is restored.
        file = open(self.filename)
        for _ in range(self.lineno):
            file.readline()
        # Finally, save the file.
        self.file = file

```

样例的使用可能是这样的
```python
>>> reader = TextReader("hello.txt")
>>> reader.readline()
'1: Hello world!'
>>> reader.readline()
'2: I am line number two.'
>>> new_reader = pickle.loads(pickle.dumps(reader))
>>> new_reader.readline()
'3: Goodbye!'
```

# 限制全局 #
默认下，反序列化会导入任何发现在pickle数据中的类和函数。对于许多应用，这样的行为是不能接受的，因为它允许unpickler导入和调用人愿意的代码。只要考虑下这个手工制作的pickle数据流在导入的时候做了什么。
```python
>>> import pickle
>>> pickle.loads(b"cos\nsystem\n(S'echo hello world'\ntR.")
hello world
0
```
在这个例子里，unpickler导入了os.system()方法，然后应用了string参数“echo hello world”。虽然，这个例子没有什么恶意，但是很难去想象这可能破坏你的系统。  
因为这个原因，你可能想要控制什么能反序列化通过定制的Unpickler.find_class().不像它的名字一样，Unpickler.find_class()可以被调用不管什么时候全局要求。因而，可以完全禁止全球化，也可以将它们限制为一个安全的子集。  
这里有一个例子，一个unpickler循序只有少量安全的来自builtins模块的类能被加载：
```python
import builtins
import io
import pickle

safe_builtins = {
    'range',
    'complex',
    'set',
    'frozenset',
    'slice',
}

class RestrictedUnpickler(pickle.Unpickler):

    def find_class(self, module, name):
        # Only allow safe classes from builtins.
        if module == "builtins" and name in safe_builtins:
            return getattr(builtins, name)
        # Forbid everything else.
        raise pickle.UnpicklingError("global '%s.%s' is forbidden" %
                                     (module, name))

def restricted_loads(s):
    """Helper function analogous to pickle.loads()."""
    return RestrictedUnpickler(io.BytesIO(s)).load()
```

一个使用我们的unpickler工作的样例的目的是：
```python
	>>> restricted_loads(pickle.dumps([1, 2, range(15)]))
	[1, 2, range(0, 15)]
	>>> restricted_loads(b"cos\nsystem\n(S'echo hello world'\ntR.")
	Traceback (most recent call last):
	  ...
	pickle.UnpicklingError: global 'os.system' is forbidden
	>>> restricted_loads(b'cbuiltins\neval\n'
	...                  b'(S\'getattr(__import__("os"), "system")'
	...                  b'("echo hello world")\'\ntR.')
	Traceback (most recent call last):
	  ...
	pickle.UnpicklingError: global 'builtins.eval' is forbidden

```
像我们的例子展示的，你需要注意什么允许被反序列化。因而，如果安全性是一个问题，你可能需要考虑替代方案，比如xmlrpc.client 中的marshalling api或者第三方的解决方法。

# 资料 #
[pickle — Python object serialization](https://docs.python.org/3.5/library/pickle.html)
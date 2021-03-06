"collections.abc" --- 容器的抽象基类
************************************

3.3 新版功能: 该模块曾是 "collections" 模块的组成部分。

**源代码：** Lib/_collections_abc.py

======================================================================

该模块定义了一些 *抽象基类*，它们可用于判断一个具体类是否具有某一特定
的接口；例如，这个类是否可哈希，或其是否为映射类。


容器抽象基类
============

这个容器模块提供了以下 *ABCs*:

+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| 抽象基类                   | 继承自                 | 抽象方法                | Mixin 方法                                           |
|============================|========================|=========================|======================================================|
| "Container"                |                        | "__contains__"          |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Hashable"                 |                        | "__hash__"              |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Iterable"                 |                        | "__iter__"              |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Iterator"                 | "Iterable"             | "__next__"              | "__iter__"                                           |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Reversible"               | "Iterable"             | "__reversed__"          |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Generator"                | "Iterator"             | "send", "throw"         | "close", "__iter__", "__next__"                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Sized"                    |                        | "__len__"               |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Callable"                 |                        | "__call__"              |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Collection"               | "Sized", "Iterable",   | "__contains__",         |                                                      |
|                            | "Container"            | "__iter__", "__len__"   |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Sequence"                 | "Reversible",          | "__getitem__",          | "__contains__", "__iter__", "__reversed__", "index", |
|                            | "Collection"           | "__len__"               | and "count"                                          |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "MutableSequence"          | "Sequence"             | "__getitem__",          | 继承自 "Sequence" 的方法，以及 "append", "reverse",  |
|                            |                        | "__setitem__",          | "extend", "pop", "remove"，和 "__iadd__"             |
|                            |                        | "__delitem__",          |                                                      |
|                            |                        | "__len__", "insert"     |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "ByteString"               | "Sequence"             | "__getitem__",          | 继承自 "Sequence" 的方法                             |
|                            |                        | "__len__"               |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Set"                      | "Collection"           | "__contains__",         | "__le__", "__lt__", "__eq__", "__ne__", "__gt__",    |
|                            |                        | "__iter__", "__len__"   | "__ge__", "__and__", "__or__", "__sub__", "__xor__", |
|                            |                        |                         | and "isdisjoint"                                     |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "MutableSet"               | "Set"                  | "__contains__",         | 继承自 "Set" 的方法以及 "clear", "pop", "remove",    |
|                            |                        | "__iter__", "__len__",  | "__ior__", "__iand__", "__ixor__"，和  "__isub__"    |
|                            |                        | "add", "discard"        |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Mapping"                  | "Collection"           | "__getitem__",          | "__contains__", "keys", "items", "values", "get",    |
|                            |                        | "__iter__", "__len__"   | "__eq__", and "__ne__"                               |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "MutableMapping"           | "Mapping"              | "__getitem__",          | 继承自 "Mapping" 的方法以及 "pop", "popitem",        |
|                            |                        | "__setitem__",          | "clear", "update"，和 "setdefault"                   |
|                            |                        | "__delitem__",          |                                                      |
|                            |                        | "__iter__", "__len__"   |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "MappingView"              | "Sized"                |                         | "__len__"                                            |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "ItemsView"                | "MappingView", "Set"   |                         | "__contains__", "__iter__"                           |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "KeysView"                 | "MappingView", "Set"   |                         | "__contains__", "__iter__"                           |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "ValuesView"               | "MappingView",         |                         | "__contains__", "__iter__"                           |
|                            | "Collection"           |                         |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Awaitable"                |                        | "__await__"             |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "Coroutine"                | "Awaitable"            | "send", "throw"         | "close"                                              |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "AsyncIterable"            |                        | "__aiter__"             |                                                      |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "AsyncIterator"            | "AsyncIterable"        | "__anext__"             | "__aiter__"                                          |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+
| "AsyncGenerator"           | "AsyncIterator"        | "asend", "athrow"       | "aclose", "__aiter__", "__anext__"                   |
+----------------------------+------------------------+-------------------------+------------------------------------------------------+

class collections.abc.Container
class collections.abc.Hashable
class collections.abc.Sized
class collections.abc.Callable

   分别提供了 "__contains__()", "__hash__()", "__len__()" 和
   "__call__()" 方法的抽象基类。

class collections.abc.Iterable

   提供了 "__iter__()" 方法的抽象基类。

   使用 "isinstance(obj, Iterable)" 可以检测一个类是否已经注册到了
   "Iterable" 或者实现了 "__iter__()" 函数，但是无法检测这个类是否能够
   使用 "__getitem__()" 方法进行迭代。检测一个对象是否是 *iterable* 的
   唯一可信赖的方法是调用 "iter(obj)"。

class collections.abc.Collection

   集合了 Sized 和 Iterable 类的抽象基类。

   3.6 新版功能.

class collections.abc.Iterator

   提供了 "__iter__()" 和 "__next__()" 方法的抽象基类。参见 *iterator*
   的定义。

class collections.abc.Reversible

   为可迭代类提供了 "__reversed__()" 方法的抽象基类。

   3.6 新版功能.

class collections.abc.Generator

   生成器类，实现了 **PEP 342** 中定义的协议，继承并扩展了迭代器，提供
   了 "send()", "throw()" 和 "close()" 方法。参见 *generator* 的定义。

   3.5 新版功能.

class collections.abc.Sequence
class collections.abc.MutableSequence
class collections.abc.ByteString

   只读且可变的序列 *sequences* 的抽象基类。

   实现笔记：一些混入（Maxin）方法比如 "__iter__()", "__reversed__()"
   和 "index()" 会重复调用底层的 "__getitem__()" 方法。因此，如果实现
   的 "__getitem__()" 是常数级访问速度，那么相应的混入方法会有一个线性
   的表现；然而，如果底层方法是线性实现（例如链表），那么混入方法将会
   是平方级的表现，这也许就需要被重构了。

   在 3.5 版更改: index() 方法支持 *stop* 和 *start* 参数。

class collections.abc.Set
class collections.abc.MutableSet

   只读且可变的集合的抽象基类。

class collections.abc.Mapping
class collections.abc.MutableMapping

   只读且可变的映射 *mappings* 的抽象基类。

class collections.abc.MappingView
class collections.abc.ItemsView
class collections.abc.KeysView
class collections.abc.ValuesView

   映射及其键和值的视图 *views* 的抽象基类。

class collections.abc.Awaitable

   为可等待对象 *awaitable* 提供的类，可以被用于 "await" 表达式中。习
   惯上必须实现 "__await__()" 方法。

   协程对象 *Coroutine* 和 "Coroutine" 抽象基类的实例都是这个抽象基类
   的实例。

   注解:

     在 CPython 里，基于生成器的协程（使用 "types.coroutine()" 或
     "asyncio.coroutine()" 包装的生成器）都是 *可等待对象*，即使他们不
     含有 "__await__()" 方法。使用 "isinstance(gencoro, Awaitable)" 来
     检测他们会返回 "False"。要使用 "inspect.isawaitable()" 来检测他们
     。

   3.5 新版功能.

class collections.abc.Coroutine

   用于协程兼容类的抽象基类。实现了如下定义在 协程对象: 里的方法：
   "send()"，"throw()" 和 "close()"。通常的实现里还需要实现
   "__await__()" 方法。所有的 "Coroutine" 实例都必须是 "Awaitable" 实
   例。参见 *coroutine* 的定义。

   注解:

     在 CPython 里，基于生成器的协程（使用 "types.coroutine()" 或
     "asyncio.coroutine()" 包装的生成器）都是 *可等待对象*，即使他们不
     含有 "__await__()" 方法。使用 "isinstance(gencoro, Coroutine)" 来
     检测他们会返回 "False"。要使用 "inspect.isawaitable()" 来检测他们
     。

   3.5 新版功能.

class collections.abc.AsyncIterable

   提供了 "__aiter__" 方法的抽象基类。参见 *asynchronous iterable* 的
   定义。

   3.5 新版功能.

class collections.abc.AsyncIterator

   提供了 "__aiter__" 和 "__anext__" 方法的抽象基类。参见
   *asynchronous iterator* 的定义。

   3.5 新版功能.

class collections.abc.AsyncGenerator

   为异步生成器类提供的抽象基类，这些类实现了定义在 **PEP 525** 和
   **PEP 492** 里的协议。

   3.6 新版功能.

这些抽象基类让我们可以确定类和示例拥有某些特定的函数，例如：

   size = None
   if isinstance(myvar, collections.abc.Sized):
       size = len(myvar)

有些抽象基类也可以用作混入类（mixin），这可以更容易地开发支持容器 API
的类。例如，要写一个支持完整 "Set" API 的类，只需要提供下面这三个方法
： "__contains__()", "__iter__()" 和 "__len__()"。抽象基类会补充上其余
的方法，比如 "__and__()" 和 "isdisjoint()":

   class ListBasedSet(collections.abc.Set):
       ''' Alternate set implementation favoring space over speed
           and not requiring the set elements to be hashable. '''
       def __init__(self, iterable):
           self.elements = lst = []
           for value in iterable:
               if value not in lst:
                   lst.append(value)

       def __iter__(self):
           return iter(self.elements)

       def __contains__(self, value):
           return value in self.elements

       def __len__(self):
           return len(self.elements)

   s1 = ListBasedSet('abcdef')
   s2 = ListBasedSet('defghi')
   overlap = s1 & s2            # The __and__() method is supported automatically

当把 "Set" 和 "MutableSet" 用作混入类时需注意：

1. 由于某些集合操作会创建新集合，默认的混入方法需要一种从可迭代对象里
   创建新实例的方法。假如其类构造函数签名形如 "ClassName(iterable)" ，
   则其会调用一个内部的类方法 "_from_iterable()"，其中调用了
   "cls(iterable)" 来生成一个新集合。如果这个 "Set" 混入类在类中被使用
   ，但其构造函数的签名却是不同的形式，那么你就需要重载
   "_from_iterable()" 方法，将其编写成一个类方法，并且它能够从可迭代对
   象参数中构造一个新实例。

2. 重载比较符时时（想必是为了速度，因为其语义都是固定的），只需要重定
   义 "__le__()" 和 "__ge__()" 函数，然后其他的操作会自动跟进。

3. 混入集合类 "Set" 提供了一个 "_hash()" 方法为集合计算哈希值，然而，
   "__hash__()" 函数却没有被定义，因为并不是所有集合都是可哈希并且不可
   变的。为了使用混入类为集合添加哈希能力，可以同时继承 "Set()" 和
   "Hashable()" 类，然后定义 "__hash__ = Set._hash"。

参见:

  * OrderedSet recipe 是基于 "MutableSet" 构建的一个示例。

  * 对于抽象基类，参见 "abc" 模块和 **PEP 3119**。

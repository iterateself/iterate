"abc" --- 抽象基类
******************

**源代码：** Lib/abc.py

======================================================================

该模块提供了在 Python 中定义 *抽象基类* (ABC) 的组件，在 **PEP 3119**
中已有概述。查看 PEP 文档了解为什么需要在 Python 中增加这个模块。（也
可查看 **PEP 3141** 以及 "numbers" 模块了解基于 ABC 的数字类型继承关系
。）

"collections" 模块中有一些派生自 ABC 的具体类；当然这些类还可以进一步
被派生。此外，"collections.abc" 子模块中有一些 ABC 可被用于测试一个类
或实例是否提供特定的接口，例如它是否可哈希或它是否为映射等。

该模块提供了一个元类 "ABCMeta"，可以用来定义抽象类，另外还提供一个工具
类 "ABC"，可以用它以继承的方式定义抽象基类。

class abc.ABC

   一个使用 "ABCMeta" 作为元类的工具类。抽象基类可以通过从 "ABC" 派生
   来简单地创建，这就避免了在某些情况下会令人混淆的元类用法，例如：

      from abc import ABC

      class MyABC(ABC):
          pass

   注意 "ABC" 的类型仍然是 "ABCMeta"，因此继承 "ABC" 仍然需要关注元类
   使用中的注意事项，比如可能会导致元类冲突的多重继承。当然你也可以直
   接使用 "ABCMeta" 作为元类来定义抽象基类，例如：

      from abc import ABCMeta

      class MyABC(metaclass=ABCMeta):
          pass

   3.4 新版功能.

class abc.ABCMeta

   用于定义抽象基类（ABC）的元类。

   使用该元类以创建抽象基类。抽象基类可以像 mix-in 类一样直接被子类继
   承。你也可以将不相关的具体类（包括内建类）和抽象基类注册为“抽象子类
   ” —— 这些类以及它们的子类会被内建函数 "issubclass()" 识别为对应的抽
   象基类的子类，但是该抽象基类不会出现在其 MRO（Method Resolution
   Order，方法解析顺序）中，抽象基类中实现的方法也不可调用（即使通过
   "super()" 调用也不行）。[1]

   使用 "ABCMeta" 作为元类创建的类含有如下方法：

   register(subclass)

      将“子类”注册为该抽象基类的“抽象子类”，例如：

         from abc import ABC

         class MyABC(ABC):
             pass

         MyABC.register(tuple)

         assert issubclass(tuple, MyABC)
         assert isinstance((), MyABC)

      在 3.3 版更改: 返回注册的子类，使其能够作为类装饰器。

      在 3.4 版更改: 你可以使用 "get_cache_token()" 函数来检测对
      "register()" 的调用。

   你也可以在虚基类中重载这个方法。

   __subclasshook__(subclass)

      （必须定义为类方法。）

      检查 *subclass* 是否是该抽象基类的子类。也就是说对于那些你希望定
      义为该抽象基类的子类的类，你不用对每个类都调用 "register()" 方法
      了，而是可以直接自定义 "issubclass" 的行为。（这个类方法是在抽象
      基类的 "__subclasscheck__()" 方法中调用的。）

      该方法必须返回 "True", "False" 或是 "NotImplemented"。如果返回
      "True"，*subclass* 就会被认为是这个抽象基类的子类。如果返回
      "False"，无论正常情况是否应该认为是其子类，统一视为不是。如果返
      回 "NotImplemented"，子类检查会按照正常机制继续执行。

   为了对这些概念做一演示，请看以下定义 ABC 的示例：

      class Foo:
          def __getitem__(self, index):
              ...
          def __len__(self):
              ...
          def get_iterator(self):
              return iter(self)

      class MyIterable(ABC):

          @abstractmethod
          def __iter__(self):
              while False:
                  yield None

          def get_iterator(self):
              return self.__iter__()

          @classmethod
          def __subclasshook__(cls, C):
              if cls is MyIterable:
                  if any("__iter__" in B.__dict__ for B in C.__mro__):
                      return True
              return NotImplemented

      MyIterable.register(Foo)

   ABC "MyIterable" 定义了标准的迭代方法 "__iter__()" 作为一个抽象方法
   。这里给出的实现仍可在子类中被调用。"get_iterator()" 方法也是
   "MyIterable" 抽象基类的一部分，但它并非必须被非抽象派生类所重载。

   The "__subclasshook__()" class method defined here says that any
   class that has an "__iter__()" method in its "__dict__" (or in that
   of one of its base classes, accessed via the "__mro__" list) is
   considered a "MyIterable" too.

   Finally, the last line makes "Foo" a virtual subclass of
   "MyIterable", even though it does not define an "__iter__()" method
   (it uses the old-style iterable protocol, defined in terms of
   "__len__()" and "__getitem__()").  Note that this will not make
   "get_iterator" available as a method of "Foo", so it is provided
   separately.

此外，"abc" 模块还提供了这些装饰器：

@abc.abstractmethod

   用于声明抽象方法的装饰器。

   使用此装饰器要求类的元类是 "ABCMeta" 或是从该类派生。一个具有派生自
   "ABCMeta" 的元类的类不可以被实例化，除非它全部的抽象方法和特征属性
   均已被重载。抽象方法可通过任何普通的“super”调用机制来调用。
   "abstractmethod()" 可被用于声明特性属性和描述器的抽象方法。

   Dynamically adding abstract methods to a class, or attempting to
   modify the abstraction status of a method or class once it is
   created, are not supported.  The "abstractmethod()" only affects
   subclasses derived using regular inheritance; "virtual subclasses"
   registered with the ABC's "register()" method are not affected.

   When "abstractmethod()" is applied in combination with other method
   descriptors, it should be applied as the innermost decorator, as
   shown in the following usage examples:

      class C(ABC):
          @abstractmethod
          def my_abstract_method(self, ...):
              ...
          @classmethod
          @abstractmethod
          def my_abstract_classmethod(cls, ...):
              ...
          @staticmethod
          @abstractmethod
          def my_abstract_staticmethod(...):
              ...

          @property
          @abstractmethod
          def my_abstract_property(self):
              ...
          @my_abstract_property.setter
          @abstractmethod
          def my_abstract_property(self, val):
              ...

          @abstractmethod
          def _get_x(self):
              ...
          @abstractmethod
          def _set_x(self, val):
              ...
          x = property(_get_x, _set_x)

   In order to correctly interoperate with the abstract base class
   machinery, the descriptor must identify itself as abstract using
   "__isabstractmethod__". In general, this attribute should be "True"
   if any of the methods used to compose the descriptor are abstract.
   For example, Python's built-in "property" does the equivalent of:

      class Descriptor:
          ...
          @property
          def __isabstractmethod__(self):
              return any(getattr(f, '__isabstractmethod__', False) for
                         f in (self._fget, self._fset, self._fdel))

   注解:

     Unlike Java abstract methods, these abstract methods may have an
     implementation. This implementation can be called via the
     "super()" mechanism from the class that overrides it.  This could
     be useful as an end-point for a super-call in a framework that
     uses cooperative multiple-inheritance.

The "abc" module also supports the following legacy decorators:

@abc.abstractclassmethod

   3.2 新版功能.

   3.3 版后已移除: It is now possible to use "classmethod" with
   "abstractmethod()", making this decorator redundant.

   A subclass of the built-in "classmethod()", indicating an abstract
   classmethod. Otherwise it is similar to "abstractmethod()".

   This special case is deprecated, as the "classmethod()" decorator
   is now correctly identified as abstract when applied to an abstract
   method:

      class C(ABC):
          @classmethod
          @abstractmethod
          def my_abstract_classmethod(cls, ...):
              ...

@abc.abstractstaticmethod

   3.2 新版功能.

   3.3 版后已移除: It is now possible to use "staticmethod" with
   "abstractmethod()", making this decorator redundant.

   A subclass of the built-in "staticmethod()", indicating an abstract
   staticmethod. Otherwise it is similar to "abstractmethod()".

   This special case is deprecated, as the "staticmethod()" decorator
   is now correctly identified as abstract when applied to an abstract
   method:

      class C(ABC):
          @staticmethod
          @abstractmethod
          def my_abstract_staticmethod(...):
              ...

@abc.abstractproperty

   3.3 版后已移除: It is now possible to use "property",
   "property.getter()", "property.setter()" and "property.deleter()"
   with "abstractmethod()", making this decorator redundant.

   A subclass of the built-in "property()", indicating an abstract
   property.

   This special case is deprecated, as the "property()" decorator is
   now correctly identified as abstract when applied to an abstract
   method:

      class C(ABC):
          @property
          @abstractmethod
          def my_abstract_property(self):
              ...

   The above example defines a read-only property; you can also define
   a read-write abstract property by appropriately marking one or more
   of the underlying methods as abstract:

      class C(ABC):
          @property
          def x(self):
              ...

          @x.setter
          @abstractmethod
          def x(self, val):
              ...

   If only some components are abstract, only those components need to
   be updated to create a concrete property in a subclass:

      class D(C):
          @C.x.setter
          def x(self, val):
              ...

"abc" 模块还提供了这些函数：

abc.get_cache_token()

   返回当前抽象基类的缓存令牌

   The token is an opaque object (that supports equality testing)
   identifying the current version of the abstract base class cache
   for virtual subclasses. The token changes with every call to
   "ABCMeta.register()" on any ABC.

   3.4 新版功能.

-[ 脚注 ]-

[1] C++ 程序员需要注意：Python 中虚基类的概念和 C++ 中的并不相同。

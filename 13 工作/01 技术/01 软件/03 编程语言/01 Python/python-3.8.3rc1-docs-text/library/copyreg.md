"copyreg" --- 注册配合 "pickle" 模块使用的函数
**********************************************

**源代码:** Lib/copyreg.py

======================================================================

"copyreg" 模块提供了可在封存特定对象时使用的一种定义函数方式。
"pickle" 和 "copy" 模块会在封存/拷贝特定对象时使用这些函数。 此模块提
供了非类对象构造器的相关配置信息。 这样的构造器可以是工厂函数或类实例
。

copyreg.constructor(object)

   将 *object* 声明为一个有效的构造器。 如果 *object* 是不可调用的（因
   而不是一个有效的构造器）则会引发 "TypeError"。

copyreg.pickle(type, function, constructor=None)

   声明该 *function* 应当被用作 *type* 类型对象的“归约函数”。
   *function* 应当返回字符串或包含两到三个元素的元组。

   如果提供了可选的 *constructor* 形参，它应当是一个可用来重建相应对象
   的可调用对象，在调用该对象时应传入由 *function* 所返回的参数元组。
   如果 *object* 是一个类或 *constructor* 是不可调用的则将引发
   "TypeError"。

   请查看 "pickle" 模块了解 *function* 和 *constructor* 所要求的接口的
   详情。 请注意一个 pickler 对象或 "pickle.Pickler" 的子类的
   "dispatch_table" 属性也可以被用来声明归约函数。


示例
====

以下示例将会显示如何注册一个封存函数，以及如何来使用它：

>>> import copyreg, copy, pickle
>>> class C(object):
...     def __init__(self, a):
...         self.a = a
...
>>> def pickle_c(c):
...     print("pickling a C instance...")
...     return C, (c.a,)
...
>>> copyreg.pickle(C, pickle_c)
>>> c = C(1)
>>> d = copy.copy(c)  
pickling a C instance...
>>> p = pickle.dumps(c)  
pickling a C instance...

"decimal" --- 十进制定点和浮点运算
**********************************

**源码：** Lib/decimal.py

======================================================================

"decimal" 模块为快速正确舍入的十进制浮点运算提供支持。 与 "float" 数据
类型相比，它具有以下几个优点：

* Decimal “基于一个浮点模型，它是为人们设计的，并且必然具有最重要的指
  导原则 —— 计算机必须提供与人们在学校学习的算法相同的算法。” —— 摘自
  十进制算术规范。

* Decimal 数字的表示是完全精确的。 相比之下，"1.1" 和 "2.2" 这样的数字
  在二进制浮点中没有精确的表示。 最终用户通常不希望 "1.1 + 2.2" 如二进
  制浮点数表示那样被显示为 "3.3000000000000003"。

* 精确性延续到算术中。 在十进制浮点数中，"0.1 + 0.1 + 0.1 - 0.3" 恰好
  等于零。 在二进制浮点数中，结果为 "5.5511151231257827e-017" 。 虽然
  接近于零，但差异妨碍了可靠的相等性检验，并且差异可能会累积。 因此，
  在具有严格相等不变量的会计应用程序中， decimal 是首选。

* 十进制模块包含有效位的概念，因此 "1.30 + 1.20" 的结果是 "2.50" 。 保
  留尾随零以表示有效位。 这是货币的惯用表示方法。乘法则沿用 “教科书“
  中：保留被乘数中的所有数字的方法。 例如， "1.3 * 1.2" 结果是 "1.56"
  而 "1.30 * 1.20" 结果是 "1.5600" 。

* 与基于硬件的二进制浮点不同，十进制模块具有用户可更改的精度（默认为28
  位），可以与给定问题所需的一样大：

  >>> from decimal import *
  >>> getcontext().prec = 6
  >>> Decimal(1) / Decimal(7)
  Decimal('0.142857')
  >>> getcontext().prec = 28
  >>> Decimal(1) / Decimal(7)
  Decimal('0.1428571428571428571428571429')

* 二进制和十进制浮点都是根据已发布的标准实现的。 虽然内置浮点类型只公
  开其功能的一小部分，但十进制模块公开了标准的所有必需部分。 在需要时
  ，程序员可以完全控制舍入和信号处理。 这包括通过使用异常来阻止任何不
  精确操作来强制执行精确算术的选项。

* 十进制模块旨在支持“无偏见，精确的非连续十进制算术（有时称为定点算术
  ）和舍入浮点算术”。 —— 摘自十进制算术规范。

模块设计以三个概念为中心：十进制数，算术上下文和信号。

十进制数是不可变的。 它有一个符号，系数数字和一个指数。 为了保持重要性
，系数数字不会截断尾随零。十进制数也包括特殊值，例如 "Infinity" ，
"-Infinity" ，和 "NaN" 。 该标准还区分 "-0" 和 "+0" 。

算术的上下文是指定精度、舍入规则、指数限制、指示操作结果的标志以及确定
符号是否被视为异常的陷阱启用器的环境。 舍入选项包括 "ROUND_CEILING" 、
"ROUND_DOWN" 、 "ROUND_FLOOR" 、 "ROUND_HALF_DOWN", "ROUND_HALF_EVEN"
、 "ROUND_HALF_UP" 、 "ROUND_UP" 以及 "ROUND_05UP".

信号是在计算过程中出现的异常条件组。 根据应用程序的需要，信号可能会被
忽略，被视为信息，或被视为异常。 十进制模块中的信号有："Clamped" 、
"InvalidOperation" 、 "DivisionByZero" 、 "Inexact" 、 "Rounded" 、
"Subnormal" 、 "Overflow" 、 "Underflow" 以及 "FloatOperation" 。

对于每个信号，都有一个标志和一个陷阱启动器。 遇到信号时，其标志设置为
1 ，然后，如果陷阱启用器设置为 1 ，则引发异常。 标志是粘性的，因此用户
需要在监控计算之前重置它们。

参见:

  * IBM的通用十进制算术规范， The General Decimal Arithmetic
    Specification.


快速入门教程
============

通常使用小数的开始是导入模块，使用 "getcontext()" 查看当前上下文，并在
必要时为精度、舍入或启用的陷阱设置新值:

   >>> from decimal import *
   >>> getcontext()
   Context(prec=28, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999,
           capitals=1, clamp=0, flags=[], traps=[Overflow, DivisionByZero,
           InvalidOperation])

   >>> getcontext().prec = 7       # Set a new precision

可以从整数、字符串、浮点数或元组构造十进制实例。 从整数或浮点构造将执
行该整数或浮点值的精确转换。 十进制数包括特殊值，例如 "NaN" 代表“非数
字”，正的和负的 "Infinity"，和 "-0"

   >>> getcontext().prec = 28
   >>> Decimal(10)
   Decimal('10')
   >>> Decimal('3.14')
   Decimal('3.14')
   >>> Decimal(3.14)
   Decimal('3.140000000000000124344978758017532527446746826171875')
   >>> Decimal((0, (3, 1, 4), -2))
   Decimal('3.14')
   >>> Decimal(str(2.0 ** 0.5))
   Decimal('1.4142135623730951')
   >>> Decimal(2) ** Decimal('0.5')
   Decimal('1.414213562373095048801688724')
   >>> Decimal('NaN')
   Decimal('NaN')
   >>> Decimal('-Infinity')
   Decimal('-Infinity')

如果 "FloatOperation" 信号被捕获，构造函数中的小数和浮点数的意外混合或
排序比较会引发异常

   >>> c = getcontext()
   >>> c.traps[FloatOperation] = True
   >>> Decimal(3.14)
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   decimal.FloatOperation: [<class 'decimal.FloatOperation'>]
   >>> Decimal('3.5') < 3.7
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   decimal.FloatOperation: [<class 'decimal.FloatOperation'>]
   >>> Decimal('3.5') == 3.5
   True

3.3 新版功能.

新 Decimal 的重要性仅由输入的位数决定。 上下文精度和舍入仅在算术运算期
间发挥作用。

   >>> getcontext().prec = 6
   >>> Decimal('3.0')
   Decimal('3.0')
   >>> Decimal('3.1415926535')
   Decimal('3.1415926535')
   >>> Decimal('3.1415926535') + Decimal('2.7182818285')
   Decimal('5.85987')
   >>> getcontext().rounding = ROUND_UP
   >>> Decimal('3.1415926535') + Decimal('2.7182818285')
   Decimal('5.85988')

如果超出了C版本的内部限制，则构造一个十进制将引发 "InvalidOperation"

   >>> Decimal("1e9999999999999999999")
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   decimal.InvalidOperation: [<class 'decimal.InvalidOperation'>]

在 3.3 版更改.

小数与 Python 的其余部分很好地交互。 这是一个小的十进制浮点飞行杂技团
：

   >>> data = list(map(Decimal, '1.34 1.87 3.45 2.35 1.00 0.03 9.25'.split()))
   >>> max(data)
   Decimal('9.25')
   >>> min(data)
   Decimal('0.03')
   >>> sorted(data)
   [Decimal('0.03'), Decimal('1.00'), Decimal('1.34'), Decimal('1.87'),
    Decimal('2.35'), Decimal('3.45'), Decimal('9.25')]
   >>> sum(data)
   Decimal('19.29')
   >>> a,b,c = data[:3]
   >>> str(a)
   '1.34'
   >>> float(a)
   1.34
   >>> round(a, 1)
   Decimal('1.3')
   >>> int(a)
   1
   >>> a * 5
   Decimal('6.70')
   >>> a * b
   Decimal('2.5058')
   >>> c % a
   Decimal('0.77')

Decimal 也可以使用一些数学函数：

>>> getcontext().prec = 28
>>> Decimal(2).sqrt()
Decimal('1.414213562373095048801688724')
>>> Decimal(1).exp()
Decimal('2.718281828459045235360287471')
>>> Decimal('10').ln()
Decimal('2.302585092994045684017991455')
>>> Decimal('10').log10()
Decimal('1')

"quantize()" 方法将数字舍入为固定指数。 此方法对于将结果舍入到固定的位
置的货币应用程序非常有用：

>>> Decimal('7.325').quantize(Decimal('.01'), rounding=ROUND_DOWN)
Decimal('7.32')
>>> Decimal('7.325').quantize(Decimal('1.'), rounding=ROUND_UP)
Decimal('8')

如上所示，"getcontext()" 函数访问当前上下文并允许更改设置。 这种方法满
足大多数应用程序的需求。

对于更高级的工作，使用 Context() 构造函数创建备用上下文可能很有用。 要
使用备用活动，请使用 "setcontext()" 函数。

根据标准，"decimal" 模块提供了两个现成的标准上下文 "BasicContext" 和
"ExtendedContext" 。 前者对调试特别有用，因为许多陷阱都已启用：

   >>> myothercontext = Context(prec=60, rounding=ROUND_HALF_DOWN)
   >>> setcontext(myothercontext)
   >>> Decimal(1) / Decimal(7)
   Decimal('0.142857142857142857142857142857142857142857142857142857142857')

   >>> ExtendedContext
   Context(prec=9, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999,
           capitals=1, clamp=0, flags=[], traps=[])
   >>> setcontext(ExtendedContext)
   >>> Decimal(1) / Decimal(7)
   Decimal('0.142857143')
   >>> Decimal(42) / Decimal(0)
   Decimal('Infinity')

   >>> setcontext(BasicContext)
   >>> Decimal(42) / Decimal(0)
   Traceback (most recent call last):
     File "<pyshell#143>", line 1, in -toplevel-
       Decimal(42) / Decimal(0)
   DivisionByZero: x / 0

上下文还具有用于监视计算期间遇到的异常情况的信号标志。 标志保持设置直
到明确清除，因此最好通过使用 "clear_flags()" 方法清除每组受监控计算之
前的标志。:

   >>> setcontext(ExtendedContext)
   >>> getcontext().clear_flags()
   >>> Decimal(355) / Decimal(113)
   Decimal('3.14159292')
   >>> getcontext()
   Context(prec=9, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999,
           capitals=1, clamp=0, flags=[Inexact, Rounded], traps=[])

*flags* 条目显示对 "Pi" 的有理逼近被舍入（超出上下文精度的数字被抛弃）
并且结果是不精确的（一些丢弃的数字不为零）。

使用上下文的 "traps" 字段中的字典设置单个陷阱：

   >>> setcontext(ExtendedContext)
   >>> Decimal(1) / Decimal(0)
   Decimal('Infinity')
   >>> getcontext().traps[DivisionByZero] = 1
   >>> Decimal(1) / Decimal(0)
   Traceback (most recent call last):
     File "<pyshell#112>", line 1, in -toplevel-
       Decimal(1) / Decimal(0)
   DivisionByZero: x / 0

大多数程序仅在程序开始时调整当前上下文一次。 并且，在许多应用程序中，
数据在循环内单个强制转换为 "Decimal" 。 通过创建上下文集和小数，程序的
大部分操作数据与其他 Python 数字类型没有区别。


Decimal 对象
============

class decimal.Decimal(value="0", context=None)

   根据 *value* 构造一个新的 "Decimal" 对象。

   *value* 可以是整数，字符串，元组，"float" ，或另一个 "Decimal" 对象
   。 如果没有给出 *value*，则返回 "Decimal('0')"。 如果 *value* 是一
   个字符串，它应该在前导和尾随空格字符以及下划线被删除之后符合十进制
   数字字符串语法:

      sign           ::=  '+' | '-'
      digit          ::=  '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
      indicator      ::=  'e' | 'E'
      digits         ::=  digit [digit]...
      decimal-part   ::=  digits '.' [digits] | ['.'] digits
      exponent-part  ::=  indicator [sign] digits
      infinity       ::=  'Infinity' | 'Inf'
      nan            ::=  'NaN' [digits] | 'sNaN' [digits]
      numeric-value  ::=  decimal-part [exponent-part] | infinity
      numeric-string ::=  [sign] numeric-value | [sign] nan

   当上面出现 "digit" 时也允许其他十进制数码。 其中包括来自各种其他语
   言系统的十进制数码（例如阿拉伯-印地语和天城文的数码）以及全宽数码
   "'\uff10'" 到 "'\uff19'"。

   如果 *value* 是一个 "tuple" ，它应该有三个组件，一个符号（ "0" 表示
   正数或 "1" 表示负数），一个数字的 "tuple" 和整数指数。 例如，
   "Decimal((0, (1, 4, 1, 4), -3))" 返回 "Decimal('1.414')"。

   如果 *value* 是 "float" ，则二进制浮点值无损地转换为其精确的十进制
   等效值。 此转换通常需要53位或更多位数的精度。 例如，
   "Decimal(float('1.1'))" 转换为``Decimal('1.10000000000000008881784
   1970012523233890533447265625')``。

   *context* 精度不会影响存储的位数。 这完全由 *value* 中的位数决定。
   例如，"Decimal('3.00000')" 记录所有五个零，即使上下文精度只有三。

   *context* 参数的目的是确定 *value* 是格式错误的字符串时该怎么做。
   如果上下文陷阱 "InvalidOperation"，则引发异常；否则，构造函数返回一
   个新的 Decimal，其值为 "NaN"。

   构造完成后， "Decimal" 对象是不可变的。

   在 3.2 版更改: 现在允许构造函数的参数为 "float" 实例。

   在 3.3 版更改: "float" 参数在设置 "FloatOperation" 陷阱时引发异常。
   默认情况下，陷阱已关闭。

   在 3.6 版更改: 允许下划线进行分组，就像代码中的整数和浮点文字一样。

   十进制浮点对象与其他内置数值类型共享许多属性，例如 "float" 和 "int"
   。 所有常用的数学运算和特殊方法都适用。 同样，十进制对象可以复制、
   pickle、打印、用作字典键、用作集合元素、比较、排序和强制转换为另一
   种类型（例如 "float" 或 "int" ）。

   算术对十进制对象和算术对整数和浮点数有一些小的差别。 当余数运算符
   "%" 应用于Decimal对象时，结果的符号是 *被除数* 的符号，而不是除数的
   符号:

      >>> (-7) % 4
      1
      >>> Decimal(-7) % Decimal(4)
      Decimal('-3')

   整数除法运算符 "//" 的行为类似，返回真商的整数部分（截断为零）而不
   是它的向下取整，以便保留通常的标识 "x == (x // y) * y + x % y":

      >>> -7 // 4
      -2
      >>> Decimal(-7) // Decimal(4)
      Decimal('-1')

   "%" 和 "//" 运算符实现了 "remainder" 和 "divide-integer" 操作（分别
   ），如规范中所述。

   十进制对象通常不能与浮点数或 "fractions.Fraction" 实例在算术运算中
   结合使用：例如,尝试将 "Decimal" 加到 "float" ，将引发 "TypeError"。
   但是，可以使用 Python 的比较运算符来比较 "Decimal" 实例 "x" 和另一
   个数字 "y" 。 这样可以避免在对不同类型的数字进行相等比较时混淆结果
   。

   在 3.2 版更改: 现在完全支持 "Decimal" 实例和其他数字类型之间的混合
   类型比较。

   除了标准的数字属性，十进制浮点对象还有许多专门的方法：

   adjusted()

      在移出系数最右边的数字之后返回调整后的指数，直到只剩下前导数字：
      "Decimal('321e+5').adjusted()" 返回 7 。 用于确定最高有效位相对
      于小数点的位置。

   as_integer_ratio()

      返回一对 "(n, d)" 整数，表示给定的 "Decimal" 实例作为分数、最简
      形式项并带有正分母:

         >>> Decimal('-3.14').as_integer_ratio()
         (-157, 50)

      转换是精确的。 在 Infinity 上引发 OverflowError ，在 NaN 上引起
      ValueError 。

   3.6 新版功能.

   as_tuple()

      返回一个 *named tuple* 表示的数字： "DecimalTuple(sign, digits,
      exponent)"。

   canonical()

      返回参数的规范编码。 目前，一个 "Decimal" 实例的编码始终是规范的
      ，因此该操作返回其参数不变。

   compare(other, context=None)

      比较两个 Decimal 实例的值。 "compare()" 返回一个 Decimal 实例，
      如果任一操作数是 NaN ，那么结果是 NaN

         a or b is a NaN  ==> Decimal('NaN')
         a < b            ==> Decimal('-1')
         a == b           ==> Decimal('0')
         a > b            ==> Decimal('1')

   compare_signal(other, context=None)

      除了所有 NaN 信号之外，此操作与 "compare()" 方法相同。 也就是说
      ，如果两个操作数都不是信令NaN，那么任何静默的 NaN 操作数都被视为
      信令NaN。

   compare_total(other, context=None)

      使用它们的抽象表示而不是它们的数值来比较两个操作数。 类似于
      "compare()" 方法，但结果给出了一个总排序 "Decimal" 实例。 两个
      "Decimal" 实例具有相同的数值但不同的表示形式在此排序中比较不相等
      ：

      >>> Decimal('12.0').compare_total(Decimal('12'))
      Decimal('-1')

      静默和发出信号的 NaN 也包括在总排序中。 这个函数的结果是
      "Decimal('0')" 如果两个操作数具有相同的表示，或是
      "Decimal('-1')" 如果第一个操作数的总顺序低于第二个操作数，或是
      "Decimal('1')" 如果第一个操作数在总顺序中高于第二个操作数。 有关
      总排序的详细信息，请参阅规范。

      此操作不受上下文影响且静默：不更改任何标志且不执行舍入。 作为例
      外，如果无法准确转换第二个操作数，则C版本可能会引发
      InvalidOperation。

   compare_total_mag(other, context=None)

      比较两个操作数使用它们的抽象表示而不是它们的值，如
      "compare_total()"，但忽略每个操作数的符号。
      "x.compare_total_mag(y)" 相当于
      "x.copy_abs().compare_total(y.copy_abs())"。

      此操作不受上下文影响且静默：不更改任何标志且不执行舍入。 作为例
      外，如果无法准确转换第二个操作数，则C版本可能会引发
      InvalidOperation。

   conjugate()

      只返回self，这种方法只符合 Decimal 规范。

   copy_abs()

      返回参数的绝对值。 此操作不受上下文影响并且是静默的：没有更改标
      志且不执行舍入。

   copy_negate()

      回到参数的否定。 此操作不受上下文影响并且是静默的：没有标志更改
      且不执行舍入。

   copy_sign(other, context=None)

      返回第一个操作数的副本，其符号设置为与第二个操作数的符号相同。
      例如：

      >>> Decimal('2.3').copy_sign(Decimal('-1.5'))
      Decimal('-2.3')

      此操作不受上下文影响且静默：不更改任何标志且不执行舍入。 作为例
      外，如果无法准确转换第二个操作数，则C版本可能会引发
      InvalidOperation。

   exp(context=None)

      返回给定数字的（自然）指数函数``e**x``的值。结果使用
      "ROUND_HALF_EVEN" 舍入模式正确舍入。

      >>> Decimal(1).exp()
      Decimal('2.718281828459045235360287471')
      >>> Decimal(321).exp()
      Decimal('2.561702493119680037517373933E+139')

   from_float(f)

      将浮点数转换为十进制数的类方法。

      注意， *Decimal.from_float(0.1)* 与 *Decimal('0.1')* 不同。 由于
      0.1 在二进制浮点中不能精确表示，因此该值存储为最接近的可表示值，
      即 *0x1.999999999999ap-4* 。 十进制的等效值是
      `0.1000000000000000055511151231257827021181583404541015625`。

      注解:

        从 Python 3.2 开始，"Decimal" 实例也可以直接从 "float" 构造。

         >>> Decimal.from_float(0.1)
         Decimal('0.1000000000000000055511151231257827021181583404541015625')
         >>> Decimal.from_float(float('nan'))
         Decimal('NaN')
         >>> Decimal.from_float(float('inf'))
         Decimal('Infinity')
         >>> Decimal.from_float(float('-inf'))
         Decimal('-Infinity')

      3.1 新版功能.

   fma(other, third, context=None)

      混合乘法加法。 返回 self*other+third ，中间乘积 self*other 没有
      舍入。

      >>> Decimal(2).fma(3, 5)
      Decimal('11')

   is_canonical()

      如果参数是规范的，则为返回 "True"，否则为 "False" 。 目前，
      "Decimal" 实例总是规范的，所以这个操作总是返回 "True" 。

   is_finite()

      如果参数是一个有限的数，则返回为 "True" ；如果参数为无穷大或 NaN
      ，则返回为 "False"。

   is_infinite()

      如果参数为正负无穷大，则返回为 "True" ，否则为 "False" 。

   is_nan()

      如果参数为 NaN （无论是否静默），则返回为 "True" ，否则为
      "False" 。

   is_normal(context=None)

      如果参数是一个 *标准的* 有限数则返回 "True"。 如果参数为零、次标
      准数、无穷大或 NaN 则返回 "False"。

   is_qnan()

      如果参数为静默 NaN，返回 "True"，否则返回 "False"。

   is_signed()

      如果参数带有负号，则返回为 "True"，否则返回 "False"。注意，0 和
      NaN 都可带有符号。

   is_snan()

      如果参数为显式 NaN，则返回 "True"，否则返回 "False"。

   is_subnormal(context=None)

      如果参数为次标准数，则返回 "True"，否则返回 "False"。

   is_zero()

      如果参数是0（正负皆可），则返回 "True"，否则返回 "False"。

   ln(context=None)

      返回操作数的自然对数（以 e 为底）。结果是使用 "ROUND_HALF_EVEN"
      舍入模式正确舍入的。

   log10(context=None)

      返回操作数的以十为底的对数。结果是使用 "ROUND_HALF_EVEN" 舍入模
      式正确舍入的。

   logb(context=None)

      对于一个非零数，返回其运算数的调整后指数作为一个 "Decimal" 实例
      。 如果运算数为零将返回 "Decimal('-Infinity')" 并且产生 the
      "DivisionByZero" 标志。如果运算数是无限大则返回
      "Decimal('Infinity')" 。

   logical_and(other, context=None)

      "logical_and()" 是需要两个 *逻辑运算数* 的逻辑运算（参考 逻辑操
      作数 ）。按位输出两运算数的 "and" 运算的结果。

   logical_invert(context=None)

      "logical_invert()" 是一个逻辑运算。结果是操作数的按位求反。

   logical_or(other, context=None)

      "logical_or()" 是需要两个 *logical operands* 的逻辑运算（请参阅
      逻辑操作数 ）。结果是两个运算数的按位的 "or" 运算。

   logical_xor(other, context=None)

      "logical_xor()" 是需要两个 *逻辑运算数* 的逻辑运算（参考 逻辑操
      作数 ）。结果是按位输出的两运算数的异或运算。

   max(other, context=None)

      像 "max(self, other)" 一样，除了在返回之前应用上下文舍入规则并且
      用信号通知或忽略 "NaN" 值（取决于上下文以及它们是发信号还是安静
      ）。

   max_mag(other, context=None)

      与 "max()" 方法相似，但是操作数使用绝对值完成比较。

   min(other, context=None)

      像 "min(self, other)" 一样，除了在返回之前应用上下文舍入规则并且
      用信号通知或忽略 "NaN" 值（取决于上下文以及它们是发信号还是安静
      ）。

   min_mag(other, context=None)

      与 "min()" 方法相似，但是操作数使用绝对值完成比较。

   next_minus(context=None)

      返回小于给定操作数的上下文中可表示的最大数字（或者当前线程的上下
      文中的可表示的最大数字如果没有给定上下文）。

   next_plus(context=None)

      返回大于给定操作数的上下文中可表示的最小数字（或者当前线程的上下
      文中的可表示的最小数字如果没有给定上下文）。

   next_toward(other, context=None)

      如果两运算数不相等，返回在第二个操作数的方向上最接近第一个操作数
      的数。如果两操作数数值上相等，返回将符号设置为与第二个运算数相同
      的第一个运算数的拷贝。

   normalize(context=None)

      通过去除尾随的零并将所有结果等于 "Decimal('0')" 的转化为
      "Decimal('0e0')" 来标准化数字。用于为等效类的属性生成规范值。比
      如， "Decimal('32.100')" 和 "Decimal('0.321000e+2')" 都被标准化
      为相同的值 "Decimal('32.1')"。

   number_class(context=None)

      返回一个字符串描述运算数的 *class* 。返回值是以下十个字符串中的
      一个。

      * ""-Infinity"" ，指示运算数为负无穷大。

      * ""-Normal"" ，指示该运算数是负正常数字。

      * ""-Subnormal"" ，指示该运算数是负的次标准数。

      * ""-Zero"" ，指示该运算数是负零。

      * ""-Zero"" ，指示该运算数是正零。

      * ""+Subnormal"" ，指示该运算数是正的次标准数。

      * ""+Normal"" ，指示该运算数是正的标准数。

      * ""+Infinity"" ，指示该运算数是正无穷。

      * ""NaN"" ，指示该运算数是肃静 NaN （非数字）。

      * ""sNaN"" ，指示该运算数是信号 NaN 。

   quantize(exp, rounding=None, context=None)

      返回的值等于舍入后的第一个运算数并且具有第二个操作数的指数。

      >>> Decimal('1.41421356').quantize(Decimal('1.000'))
      Decimal('1.414')

      与其他运算不同，如果量化运算后的系数长度大于精度，那么会发出一个
      "InvalidOperation" 信号。这保证了除非有一个错误情况，量化指数恒
      等于右手运算数的指数。

      与其他运算不同，量化永不信号下溢，即使结果不正常且不精确。

      如果第二个运算数的指数大于第一个运算数的指数那或许需要舍入。在这
      种情况下，舍入模式由给定 "rounding" 参数决定，其余的由给定
      "context" 参数决定；如果参数都未给定，使用当前线程上下文的舍入模
      式。

      每当结果的指数大于 "Emax" 或小于 "Etiny" 就会返回错误。

   radix()

      返回 "Decimal(10)"，即 "Decimal" 类进行所有算术运算所用的数制（
      基数）。 这是为保持与规范描述的兼容性而加入的。

   remainder_near(other, context=None)

      返回 *self* 除以 *other* 的余数。 这与 "self % other" 的区别在于
      所选择的余数要使其绝对值最小化。 更准确地说，返回值为 "self - n
      * other" 其中 "n" 是最接近 "self / other" 的实际值的整数，并且如
      果两个整数与实际值的差相等则会选择其中的偶数。

      如果结果为零则其符号将为 *self* 的符号。

      >>> Decimal(18).remainder_near(Decimal(10))
      Decimal('-2')
      >>> Decimal(25).remainder_near(Decimal(10))
      Decimal('5')
      >>> Decimal(35).remainder_near(Decimal(10))
      Decimal('-5')

   rotate(other, context=None)

      返回对第一个操作数的数码按第二个操作数所指定的数量进行轮转的结果
      。 第二个操作数必须为 -precision 至 precision 精度范围内的整数。
      第二个操作数的绝对值给出要轮转的位数。 如果第二个操作数为正值则
      向左轮转；否则向右轮转。 如有必要第一个操作数的系数会在左侧填充
      零以达到 precision 所指定的长度。 第一个操作数的符号和指数保持不
      变。

   same_quantum(other, context=None)

      检测自身与 other 是否具有相同的指数或是否均为 "NaN"。

      此操作不受上下文影响且静默：不更改任何标志且不执行舍入。 作为例
      外，如果无法准确转换第二个操作数，则C版本可能会引发
      InvalidOperation。

   scaleb(other, context=None)

      返回第一个操作数使用第二个操作数对指数进行调整的结果。 等价于返
      回第一个操作数乘以 "10**other" 的结果。 第二个操作数必须为整数。

   shift(other, context=None)

      返回第一个操作数的数码按第二个操作数所指定的数量进行移位的结果。
      第二个操作数必须为 -precision 至 precision 范围内的整数。 第二个
      操作数的绝对值给出要移动的位数。 如果第二个操作数为正值则向左移
      位；否则向右移位。 移入系数的数码为零。 第一个操作数的符号和指数
      保持不变。

   sqrt(context=None)

      返回参数的平方根精确到完整精度。

   to_eng_string(context=None)

      转换为字符串，如果需要指数则会使用工程标注法。

      工程标注法的指数是 3 的倍数。 这会在十进制位的左边保留至多 3 个
      数码，并可能要求添加一至两个末尾零。

      例如，此方法会将 "Decimal('123E+1')" 转换为 "Decimal('1.23E+3')"
      。

   to_integral(rounding=None, context=None)

      与 "to_integral_value()" 方法相同。 保留 "to_integral" 名称是为
      了与旧版本兼容。

   to_integral_exact(rounding=None, context=None)

      舍入到最接近的整数，发出信号 "Inexact" 或者如果发生舍入则相应地
      发出信号 "Rounded"。 如果给出 "rounding" 形参则由其确定舍入模式
      ，否则由给定的 "context" 来确定。 如果没有给定任何形参则会使用当
      前上下文的舍入模式。

   to_integral_value(rounding=None, context=None)

      舍入到最接近的整数而不发出 "Inexact" 或 "Rounded" 信号。 如果给
      出 *rounding* 则会应用其所指定的舍入模式；否则使用所提供的
      *context* 或当前上下文的舍入方法。


逻辑操作数
----------

"logical_and()", "logical_invert()", "logical_or()" 和 "logical_xor()"
方法期望其参数为 *逻辑操作数*。 *逻辑操作数* 是指数位与符号位均为零的
"Decimal" 实例，并且其数字位均为 "0" 或 "1"。


上下文对象
==========

上下文是算术运算所在的环境。 它们管理精度、设置舍入规则、确定将哪些信
号视为异常，并限制指数的范围。

每个线程都有自己的当前上下文，可使用 "getcontext()" 和 "setcontext()"
函数来读取或修改：

decimal.getcontext()

   返回活动线程的当前上下文。

decimal.setcontext(c)

   将活动线程的当前上下文设为 *c*。

你也可以使用 "with" 语句和 "localcontext()" 函数来临时改变活动上下文。

decimal.localcontext(ctx=None)

   返回一个上下文管理器，它将在进入 with 语句时将活动线程的当前上下文
   设为 *ctx* 的一个副本并在退出 with 语句时恢复之前的上下文。 如果未
   指定上下文，则会使用当前上下文的一个副本。

   例如，以下代码会将当前 decimal 精度设为 42 位，执行一个运算，然后自
   动恢复之前的上下文:

      from decimal import localcontext

      with localcontext() as ctx:
          ctx.prec = 42   # Perform a high precision calculation
          s = calculate_something()
      s = +s  # Round the final result back to the default precision

新的上下文也可使用下述的 "Context" 构造器来创建。 此外，模块还提供了三
种预设的上下文:

class decimal.BasicContext

   这是由通用十进制算术规范描述所定义的标准上下文。 精度设为九。 舍入
   设为 "ROUND_HALF_UP"。 清除所有旗标。 启用所有陷阱（视为异常），但
   "Inexact", "Rounded" 和 "Subnormal" 除外。

   由于启用了许多陷阱，此上下文适用于进行调试。

class decimal.ExtendedContext

   这是由通用十进制算术规范描述所定义的标准上下文。 精度设为九。 舍入
   设为 "ROUND_HALF_EVEN"。 清除所有旗标。 不启用任何陷阱（因此在计算
   期间不会引发异常）。

   由于禁用了陷阱，此上下文适用于希望结果值为 "NaN" 或 "Infinity" 而不
   是引发异常的应用。 这允许应用在出现当其他情况下会中止程序的条件时仍
   能完成运行。

class decimal.DefaultContext

   此上下文被 "Context" 构造器用作新上下文的原型。 改变一个字段（例如
   精度）的效果将是改变 "Context" 构造器所创建的新上下文的默认值。

   此上下文最适用于多线程环境。 在线程开始前改变一个字段具有设置全系统
   默认值的效果。 不推荐在线程开始后改变字段，因为这会要求线程同步避免
   竞争条件。

   在单线程环境中，最好完全不使用此上下文。 而是简单地电显式创建上下文
   ，具体如下所述。

   默认值为 "prec"="28", "rounding"="ROUND_HALF_EVEN"，并为
   "Overflow", "InvalidOperation" 和 "DivisionByZero" 启用陷阱。

在已提供的三种上下文之外，还可以使用 "Context" 构造器创建新的上下文。

class decimal.Context(prec=None, rounding=None, Emin=None, Emax=None, capitals=None, clamp=None, flags=None, traps=None)

   创建一个新上下文。 如果某个字段未指定或为 "None"，则从
   "DefaultContext" 拷贝默认值。 如果 *flags* 字段未指定或为 "None"，
   则清空所有旗标。

   *prec* 为一个 ["1", "MAX_PREC"] 范围内的整数，用于设置该上下文中算
   术运算的精度。

   *rounding* 选项应为 Rounding Modes 小节中列出的常量之一。

   *traps* 和 *flags* 字段列出要设置的任何信号。 通常，新上下文应当只
   设置 traps 而让 flags 为空。

   *Emin* 和 *Emax* 字段给定指数所允许的外部上限。 *Emin* 必须在
   ["MIN_EMIN", "0"] 范围内，*Emax* 在 ["0", "MAX_EMAX"] 范围内。

   *capitals* 字段为 "0" 或 "1" (默认值)。 如果设为 "1"，指数将附带打
   印大写的 "E"；其他情况则将使用小写的 "e": "Decimal('6.02e+23')"。

   *clamp* 字段为 "0" (默认值) 或 "1"。 如果设为 "1"，则 "Decimal" 实
   例的指数 "e" 的表示范围在此上下文中将严格限制为 "Emin - prec + 1 <=
   e <= Emax - prec + 1"。 如果 *clamp* 为 "0" 则将适用较弱的条件:
   "Decimal" 实例调整后的指数最大值为 "Emax"。 当 *clamp* 为 "1" 时，
   一个较大的普通数值将在可能的情况下减小其指数并为其系统添加相应数量
   的零，以便符合指数值限制；这可以保持数字值但会丢失有效末尾零的信息
   。 例如:

      >>> Context(prec=6, Emax=999, clamp=1).create_decimal('1.23e999')
      Decimal('1.23000E+999')

   *clamp* 值为 "1" 时即允许与在 IEEE 754 中描述的固定宽度十进制交换格
   式保持兼容性。

   "Context" 类定义了几种通用方法以及大量直接在给定上下文中进行算术运
   算的方法。 此外，对于上述的每种 "Decimal" 方法（不包括 "adjusted()"
   和 "as_tuple()" 方法）都有一个相应的 "Context" 方法。 例如，对于一
   个 "Context" 的实例 "C" 和 "Decimal" 的实例 "x"，"C.exp(x)" 就等价
   于 "x.exp(context=C)"。 每个 "Context" 方法都接受一个 Python 整数（
   即 "int" 的实例）在任何接受 Decimal 的实例的地方使用。

   clear_flags()

      将所有旗标重置为 "0"。

   clear_traps()

      将所有陷阱重置为零 "0"。

      3.3 新版功能.

   copy()

      返回上下文的一个副本。

   copy_decimal(num)

      返回 Decimal 实例 num 的一个副本。

   create_decimal(num)

      基于 *num* 创建一个新 Decimal 实例但使用 *self* 作为上下文。 与
      "Decimal" 构造器不同，该上下文的精度、舍入方法、旗标和陷阱会被应
      用于转换过程。

      此方法很有用处，因为常量往往被给予高于应用所需的精度。 另一个好
      处在于立即执行舍入可以消除超出当前精度的数位所导致的意外效果。
      在下面的示例中，使用未舍入的输入意味着在总和中添加零会改变结果：

         >>> getcontext().prec = 3
         >>> Decimal('3.4445') + Decimal('1.0023')
         Decimal('4.45')
         >>> Decimal('3.4445') + Decimal(0) + Decimal('1.0023')
         Decimal('4.44')

      此方法实现了 IBM 规格描述中的转换为数字操作。 如果参数为字符串，
      则不允许有开头或末尾的空格或下划线。

   create_decimal_from_float(f)

      基于浮点数 *f* 创建一个新的 Decimal 实例，但会使用 *self* 作为上
      下文来执行舍入。 与 "Decimal.from_float()" 类方法不同，上下文的
      精度、舍入方法、旗标和陷阱会应用到转换中。

         >>> context = Context(prec=5, rounding=ROUND_DOWN)
         >>> context.create_decimal_from_float(math.pi)
         Decimal('3.1415')
         >>> context = Context(prec=5, traps=[Inexact])
         >>> context.create_decimal_from_float(math.pi)
         Traceback (most recent call last):
             ...
         decimal.Inexact: None

      3.1 新版功能.

   Etiny()

      返回一个等于 "Emin - prec + 1" 的值即次标准化结果中的最小指数值
      。 当发生向下溢出时，指数会设为 "Etiny"。

   Etop()

      返回一个等于 "Emax - prec + 1" 的值。

   使用 decimal 的通常方式是创建 "Decimal" 实例然后对其应用算术运算,这
   些运算发生在活动线程的当前上下文中。 一种替代方式则是使用上下文的方
   法在特定上下文中进行计算。 这些方法类似于 "Decimal" 类的方法，在此
   仅简单地重新列出。

   abs(x)

      返回 *x* 的绝对值。

   add(x, y)

      返回 *x* 与 *y* 的和。

   canonical(x)

      返回相同的 Decimal 对象 *x*。

   compare(x, y)

      对 *x* 与 *y* 进行数值比较。

   compare_signal(x, y)

      对两个操作数进行数值比较。

   compare_total(x, y)

      对两个操作数使用其抽象表示进行比较。

   compare_total_mag(x, y)

      对两个操作数使用其抽象表示进行比较，忽略符号。

   copy_abs(x)

      返回 *x* 的副本，符号设为 0。

   copy_negate(x)

      返回 *x* 的副本，符号取反。

   copy_sign(x, y)

      从 *y* 拷贝符号至 *x*。

   divide(x, y)

      返回 *x* 除以 *y* 的结果。

   divide_int(x, y)

      返回 *x* 除以 *y* 的结果，截短为整数。

   divmod(x, y)

      两个数字相除并返回结果的整数部分。

   exp(x)

      返回 *e ** x*。

   fma(x, y, z)

      返回 *x* 乘以 *y* 再加 *z* 的结果。

   is_canonical(x)

      如果 *x* 是规范的则返回 "True"；否则返回 "False"。

   is_finite(x)

      如果 *x* 为有限的则返回``True``；否则返回 "False"。

   is_infinite(x)

      如果 *x* 是无限的则返回 "True"；否则返回 "False"。

   is_nan(x)

      如果 *x* 是 qNaN 或 sNaN 则返回 "True"；否则返回 "False"。

   is_normal(x)

      如果 *x* 是标准数则返回 "True"；否则返回 "False"。

   is_qnan(x)

      如果 *x* 是静默 NaN 则返回 "True"；否则返回 "False"。

   is_signed(x)

      *x* 是负数则返回 "True"；否则返回 "False"。

   is_snan(x)

      如果 *x* 是显式 NaN 则返回 "True"；否则返回 "False"。

   is_subnormal(x)

      如果 *x* 是次标准数则返回 "True"；否则返回 "False"。

   is_zero(x)

      如果 *x* 为零则返回 "True"；否则返回 "False"。

   ln(x)

      返回 *x* 的自然对数（以 e 为底）。

   log10(x)

      返回 *x* 的以 10 为底的对数。

   logb(x)

      返回操作数的 MSD 等级的指数。

   logical_and(x, y)

      在操作数的每个数位间应用逻辑运算 *and*。

   logical_invert(x)

      反转 *x* 中的所有数位。

   logical_or(x, y)

      在操作数的每个数位间应用逻辑运算 *or*。

   logical_xor(x, y)

      在操作数的每个数位间应用逻辑运算 *xor*。

   max(x, y)

      对两个值执行数字比较并返回其中的最大值。

   max_mag(x, y)

      对两个值执行忽略正负号的数字比较。

   min(x, y)

      对两个值执行数字比较并返回其中的最小值。

   min_mag(x, y)

      对两个值执行忽略正负号的数字比较。

   minus(x)

      对应于 Python 中的单目前缀取负运算符执行取负操作。

   multiply(x, y)

      返回 *x* 和 *y* 的积。

   next_minus(x)

      返回小于 *x* 的最大数字表示形式。

   next_plus(x)

      返回大于 *x* 的最小数字表示形式。

   next_toward(x, y)

      返回 *x* 趋向于 *y* 的最接近的数字。

   normalize(x)

      将 *x* 改写为最简形式。

   number_class(x)

      返回 *x* 的类的表示。

   plus(x)

      对应于 Python 中的单目前缀取正运算符执行取正操作。 此操作将应用
      上下文精度和舍入，因此它 *不是* 标识运算。

   power(x, y, modulo=None)

      返回 "x" 的 "y" 次方，如果给出了模数 "modulo" 则取其余数。

      如为两个参数则计算 "x**y"。 如果 "x" 为负值则 "y" 必须为整数。
      除非 "y" 为整数且结果为有限值并可在 'precision' 位内精确表示否则
      结果将是不精确的。 上下文的舍入模式将被使用。 结果在 Python 版中
      总是会被正确地舍入。

      在 3.3 版更改: C 模块计算 "power()" 时会使用已正确舍入的 "exp()"
      和 "ln()" 函数。 结果是经过良好定义的，但仅限于“几乎总是正确地舍
      入”。

      带有三个参数时，计算 "(x**y) % modulo"。 对于三个参数的形式，参
      数将会应用以下限制：

         * 三个参数必须都是整数

         * "y" 必须是非负数

         * "x" 或 "y" 至少有一个不为零

         * "modulo" 必须不为零且至多有 'precision' 位

      来自 "Context.power(x, y, modulo)" 的结果值等于使用无限精度计算
      "(x**y) % modulo" 所得到的值，但其计算过程更高效。 结果的指数为
      零，无论 "x", "y" 和 "modulo" 的指数是多少。 结果值总是完全精确
      的。

   quantize(x, y)

      返回的值等于 *x* (舍入后)，并且指数为 *y*。

   radix()

      恰好返回 10，因为这是 Decimal 对象 :)

   remainder(x, y)

      返回整除所得到的余数。

      结果的符号，如果不为零，则与原始除数的符号相同。

   remainder_near(x, y)

      返回 "x - y * n"，其中 *n* 为最接近 "x / y" 实际值的整数（如结果
      为 0 则其符号将与 *x* 的符号相同）。

   rotate(x, y)

      返回 *x* 翻转 *y* 次的副本。

   same_quantum(x, y)

      如果两个操作数具有相同的指数则返回 "True"。

   scaleb(x, y)

      返回第一个操作数添加第二个值的指数后的结果。

   shift(x, y)

      返回 *x* 变换 *y* 次的副本。

   sqrt(x)

      非负数基于上下文精度的平方根。

   subtract(x, y)

      返回 *x* 和 *y* 的差。

   to_eng_string(x)

      转换为字符串，如果需要指数则会使用工程标注法。

      工程标注法的指数是 3 的倍数。 这会在十进制位的左边保留至多 3 个
      数码，并可能要求添加一至两个末尾零。

   to_integral_exact(x)

      舍入到一个整数。

   to_sci_string(x)

      使用科学计数法将一个数字转换为字符串。


常数
====

本节中的常量仅与 C 模块相关。 它们也被包含在纯 Python 版本以保持兼容性
。

+-----------------------+-----------------------+---------------------------------+
|                       | 32位                  | 64位                            |
|=======================|=======================|=================================|
| decimal.MAX_PREC      | "425000000"           | "999999999999999999"            |
+-----------------------+-----------------------+---------------------------------+
| decimal.MAX_EMAX      | "425000000"           | "999999999999999999"            |
+-----------------------+-----------------------+---------------------------------+
| decimal.MIN_EMIN      | "-425000000"          | "-999999999999999999"           |
+-----------------------+-----------------------+---------------------------------+
| decimal.MIN_ETINY     | "-849999999"          | "-1999999999999999997"          |
+-----------------------+-----------------------+---------------------------------+

decimal.HAVE_THREADS

   该值为 "True"。 已弃用，因为 Python 现在总是启用线程。

3.9 版后已移除.

decimal.HAVE_CONTEXTVAR

   默认值为 "True"。 如果 Python 的编译带有 "--without-decimal-
   contextvar" 选项，则 C 版本会使用线程局部而不是协程局部上下文并且该
   值为 "False"。 这在某些嵌套上下文场景中将稍快一些。

3.9 新版功能: 向下移植到 3.7 和 3.8


舍入模式
========

decimal.ROUND_CEILING

   舍入方向为 "Infinity"。

decimal.ROUND_DOWN

   舍入方向为零。

decimal.ROUND_FLOOR

   舍入方向为 "-Infinity"。

decimal.ROUND_HALF_DOWN

   舍入到最接近的数，同样接近则舍入方向为零。

decimal.ROUND_HALF_EVEN

   舍入到最接近的数，同样接近则舍入到最接近的偶数。

decimal.ROUND_HALF_UP

   舍入到最接近的数，同样接近则舍入到零的反方向。

decimal.ROUND_UP

   舍入到零的反方向。

decimal.ROUND_05UP

   如果最后一位朝零的方向舍入后为 0 或 5 则舍入到零的反方向；否则舍入
   方向为零。


信号
====

信号代表在计算期间引发的条件。 每个信号对应于一个上下文旗标和一个上下
文陷阱启用器。

上下文旗标将在遇到特定条件时被设定。 在完成计算之后，将为了获得信息而
检测旗标（例如确定计算是否精确）。 在检测旗标后，请确保在开始下一次计
算之前清除所有旗标。

如果为信号设定了上下文的陷阱启用器，则条件会导致特定的 Python 异常被引
发。 举例来说，如果设定了 "DivisionByZero" 陷阱，则当遇到此条件时就将
引发 "DivisionByZero" 异常。

class decimal.Clamped

   修改一个指数以符合表示限制。

   通常，限位将在一个指数超出上下文的 "Emin" 和 "Emax" 限制时发生。 在
   可能的情况下，会通过给系数添加零来将指数缩减至符合限制。

class decimal.DecimalException

   其他信号的基类，并且也是 "ArithmeticError" 的一个子类。

class decimal.DivisionByZero

   非无限数被零除的信号。

   可在除法、取余队法或对一个数求负数次幂时发生。 如果此信号未被陷阱捕
   获，则返回 "Infinity" 或 "-Infinity" 并且由对计算的输入来确定正负符
   号。

class decimal.Inexact

   表明发生了舍入且结果是不精确的。

   有非零数位在舍入期间被丢弃的信号。 舍入结果将被返回。 此信号旗标或
   陷阱被用于检测结果不精确的情况。

class decimal.InvalidOperation

   执行了一个无效的操作。

   表明请求了一个无意义的操作。 如未被陷阱捕获则返回 "NaN"。 可能的原
   因包括:

      Infinity - Infinity
      0 * Infinity
      Infinity / Infinity
      x % 0
      Infinity % x
      sqrt(-x) and x > 0
      0 ** 0
      x ** (non-integer)
      x ** Infinity

class decimal.Overflow

   数值的溢出。

   表明在发生舍入之后的指数大于 "Emax"。 如果未被陷阱捕获，则结果将取
   决于舍入模式，或者向下舍入为最大的可表示有限数，或者向上舍入为
   "Infinity"。 无论哪种情况，都将引发 "Inexact" 和 "Rounded" 信号。

class decimal.Rounded

   发生了舍入，但或许并没有信息丢失。

   一旦舍入丢弃了数位就会发出此信号；即使被丢弃的数位是零 (例如将
   "5.00" 舍入为 "5.0")。 如果未被陷阱捕获，则不经修改地返回结果。 此
   信号用于检测有效位数的丢弃。

class decimal.Subnormal

   在舍入之前指数低于 "Emin"。

   当操作结果是次标准数（即指数过小）时就会发出此信号。 如果未被陷阱捕
   获，则不经修改过返回结果。

class decimal.Underflow

   数字向下溢出导致结果舍入到零。

   当一个次标准数结果通过舍入转为零时就会发出此信号。 同时还将引发
   "Inexact" 和 "Subnormal" 信号。

class decimal.FloatOperation

   为 float 和 Decimal 的混合启用更严格的语义。

   如果信号未被捕获（默认），则在 "Decimal" 构造器、"create_decimal()"
   和所有比较运算中允许 float 和 Decimal 的混合。 转换和比较都是完全精
   确的。 发生的任何混合运算都将通过在上下文旗标中设置
   "FloatOperation" 来静默地记录。 通过 "from_float()" 或
   "create_decimal_from_float()" 进行显式转换则不会设置旗标。

   在其他情况下（即信号被捕获），则只静默执行相等性比较和显式转换。 所
   有其他混合运算都将引发 "FloatOperation"。

以下表格总结了信号的层级结构:

   exceptions.ArithmeticError(exceptions.Exception)
       DecimalException
           Clamped
           DivisionByZero(DecimalException, exceptions.ZeroDivisionError)
           Inexact
               Overflow(Inexact, Rounded)
               Underflow(Inexact, Rounded, Subnormal)
           InvalidOperation
           Rounded
           Subnormal
           FloatOperation(DecimalException, exceptions.TypeError)


浮点数说明
==========


通过提升精度来解决舍入错误
--------------------------

使用十进制浮点数可以消除十进制表示错误（即能够完全精确地表示 "0.1" 这
样的数）；然而，某些运算在非零数位超出给定的精度时仍然可能导致舍入错误
。

舍入错误的影响可能因接近相互抵销的加减运算被放大从而导致丢失有效位。
Knuth 提供了两个指导性示例，其中出现了精度不足的浮点算术舍入，导致加法
的交换律和分配律被打破：

   # Examples from Seminumerical Algorithms, Section 4.2.2.
   >>> from decimal import Decimal, getcontext
   >>> getcontext().prec = 8

   >>> u, v, w = Decimal(11111113), Decimal(-11111111), Decimal('7.51111111')
   >>> (u + v) + w
   Decimal('9.5111111')
   >>> u + (v + w)
   Decimal('10')

   >>> u, v, w = Decimal(20000), Decimal(-6), Decimal('6.0000003')
   >>> (u*v) + (u*w)
   Decimal('0.01')
   >>> u * (v+w)
   Decimal('0.0060000')

"decimal" 模块则可以通过充分地扩展精度来避免有效位的丢失：

   >>> getcontext().prec = 20
   >>> u, v, w = Decimal(11111113), Decimal(-11111111), Decimal('7.51111111')
   >>> (u + v) + w
   Decimal('9.51111111')
   >>> u + (v + w)
   Decimal('9.51111111')
   >>>
   >>> u, v, w = Decimal(20000), Decimal(-6), Decimal('6.0000003')
   >>> (u*v) + (u*w)
   Decimal('0.0060000')
   >>> u * (v+w)
   Decimal('0.0060000')


特殊的值
--------

"decimal" 模块的数字系统提供了一些特殊的值，包括 "NaN", "sNaN",
"-Infinity", "Infinity" 以及两种零值 "+0" 和 "-0"。

无穷大可以使用 "Decimal('Infinity')" 来构建。 它们也可以在不捕获
"DivisionByZero" 信号捕获时通过除以零来产生。 类似地，当不捕获
"Overflow" 信号时，也可以通过舍入到超出最大可表示数字限制的方式产生无
穷大的结果。

无穷大是有符号的（仿射）并可用于算术运算，它们会被当作极其巨大的不确定
数字来处理。 例如，无穷大加一个常量结果也将为无穷大。

某些不存在有效结果的运算将会返回 "NaN"，或者如果捕获了
"InvalidOperation" 信号则会引发一个异常。 例如，"0/0" 会返回 "NaN" 表
示结果“不是一个数字”。 这样的 "NaN" 是静默产生的，并且在产生之后参与其
它计算时总是会得到 "NaN" 的结果。 这种行为对于偶而缺少输入的各类计算都
很有用处 --- 它允许在将特定结果标记为无效的同时让计算继续运行。

另一种变体形式是 "sNaN"，它在每次运算后会发出信号而不是保持静默。 当对
于无效结果需要中断计算进行特别处理时，这是一个很有用的返回值。

Python 中比较运算符的行为在涉及 "NaN" 时可能会令人有点惊讶。 相等性检
测在操作数中有静默型或信号型 "NaN" 时总是会返回 "False" (即使是执行
"Decimal('NaN')==Decimal('NaN')")，而不等性检测总是会返回 "True"。 当
尝试使用 "<", "<=", ">" 或 ">=" 运算符中的任何一个来比较两个 Decimal
值时，如果运算数中有 "NaN" 则将引发 "InvalidOperation" 信号，如果此信
号未被捕获则将返回 "False"。 请注意通用十进制算术规范并未规定直接比较
行为；这些涉及 "NaN" 的比较规则来自于 IEEE 854 标准 (见第 5.7 节表 3)
。 要确保严格符合标准，请改用 "compare()" 和 "compare-signal()" 方法。

有符号零值可以由向下溢出的运算产生。 它们保留符号是为了让运算结果能以
更高的精度传递。 由于它们的大小为零，正零和负零会被视为相等，且它们的
符号具有信息。

在这两个不相同但却相等的有符号零之外，还存在几种零的不同表示形式，它们
的精度不同但值也都相等。 这需要一些时间来逐渐适应。 对于习惯了标准浮点
表示形式的眼睛来说，以下运算返回等于零的值并不是显而易见的：

>>> 1 / Decimal('Infinity')
Decimal('0E-1000026')


使用线程
========

"getcontext()" 函数会为每个线程访问不同的 "Context" 对象。 具有单独线
程上下文意味着线程可以修改上下文 (例如 "getcontext().prec=10") 而不影
响其他线程。

类似的 "setcontext()" 会为当前上下文的目标自动赋值。

如果在调用 "setcontext()" 之前调用了 "getcontext()"，则 "getcontext()"
将自动创建一个新的上下文在当前线程中使用。

新的上下文拷贝自一个名为 *DefaultContext* 的原型上下文。 要控制默认值
以便每个线程在应用运行期间都使用相同的值，可以直接修改
*DefaultContext* 对象。 这应当在任何线程启动 *之前* 完成以使得调用
"getcontext()" 的线程之间不会产生竞争条件。 例如:

   # Set applicationwide defaults for all threads about to be launched
   DefaultContext.prec = 12
   DefaultContext.rounding = ROUND_DOWN
   DefaultContext.traps = ExtendedContext.traps.copy()
   DefaultContext.traps[InvalidOperation] = 1
   setcontext(DefaultContext)

   # Afterwards, the threads can be started
   t1.start()
   t2.start()
   t3.start()
    . . .


例程
====

以下是一些用作工具函数的例程，它们演示了使用 "Decimal" 类的各种方式:

   def moneyfmt(value, places=2, curr='', sep=',', dp='.',
                pos='', neg='-', trailneg=''):
       """Convert Decimal to a money formatted string.

       places:  required number of places after the decimal point
       curr:    optional currency symbol before the sign (may be blank)
       sep:     optional grouping separator (comma, period, space, or blank)
       dp:      decimal point indicator (comma or period)
                only specify as blank when places is zero
       pos:     optional sign for positive numbers: '+', space or blank
       neg:     optional sign for negative numbers: '-', '(', space or blank
       trailneg:optional trailing minus indicator:  '-', ')', space or blank

       >>> d = Decimal('-1234567.8901')
       >>> moneyfmt(d, curr='$')
       '-$1,234,567.89'
       >>> moneyfmt(d, places=0, sep='.', dp='', neg='', trailneg='-')
       '1.234.568-'
       >>> moneyfmt(d, curr='$', neg='(', trailneg=')')
       '($1,234,567.89)'
       >>> moneyfmt(Decimal(123456789), sep=' ')
       '123 456 789.00'
       >>> moneyfmt(Decimal('-0.02'), neg='<', trailneg='>')
       '<0.02>'

       """
       q = Decimal(10) ** -places      # 2 places --> '0.01'
       sign, digits, exp = value.quantize(q).as_tuple()
       result = []
       digits = list(map(str, digits))
       build, next = result.append, digits.pop
       if sign:
           build(trailneg)
       for i in range(places):
           build(next() if digits else '0')
       if places:
           build(dp)
       if not digits:
           build('0')
       i = 0
       while digits:
           build(next())
           i += 1
           if i == 3 and digits:
               i = 0
               build(sep)
       build(curr)
       build(neg if sign else pos)
       return ''.join(reversed(result))

   def pi():
       """Compute Pi to the current precision.

       >>> print(pi())
       3.141592653589793238462643383

       """
       getcontext().prec += 2  # extra digits for intermediate steps
       three = Decimal(3)      # substitute "three=3.0" for regular floats
       lasts, t, s, n, na, d, da = 0, three, 3, 1, 0, 0, 24
       while s != lasts:
           lasts = s
           n, na = n+na, na+8
           d, da = d+da, da+32
           t = (t * n) / d
           s += t
       getcontext().prec -= 2
       return +s               # unary plus applies the new precision

   def exp(x):
       """Return e raised to the power of x.  Result type matches input type.

       >>> print(exp(Decimal(1)))
       2.718281828459045235360287471
       >>> print(exp(Decimal(2)))
       7.389056098930650227230427461
       >>> print(exp(2.0))
       7.38905609893
       >>> print(exp(2+0j))
       (7.38905609893+0j)

       """
       getcontext().prec += 2
       i, lasts, s, fact, num = 0, 0, 1, 1, 1
       while s != lasts:
           lasts = s
           i += 1
           fact *= i
           num *= x
           s += num / fact
       getcontext().prec -= 2
       return +s

   def cos(x):
       """Return the cosine of x as measured in radians.

       The Taylor series approximation works best for a small value of x.
       For larger values, first compute x = x % (2 * pi).

       >>> print(cos(Decimal('0.5')))
       0.8775825618903727161162815826
       >>> print(cos(0.5))
       0.87758256189
       >>> print(cos(0.5+0j))
       (0.87758256189+0j)

       """
       getcontext().prec += 2
       i, lasts, s, fact, num, sign = 0, 0, 1, 1, 1, 1
       while s != lasts:
           lasts = s
           i += 2
           fact *= i * (i-1)
           num *= x * x
           sign *= -1
           s += num / fact * sign
       getcontext().prec -= 2
       return +s

   def sin(x):
       """Return the sine of x as measured in radians.

       The Taylor series approximation works best for a small value of x.
       For larger values, first compute x = x % (2 * pi).

       >>> print(sin(Decimal('0.5')))
       0.4794255386042030002732879352
       >>> print(sin(0.5))
       0.479425538604
       >>> print(sin(0.5+0j))
       (0.479425538604+0j)

       """
       getcontext().prec += 2
       i, lasts, s, fact, num, sign = 1, 0, x, 1, x, 1
       while s != lasts:
           lasts = s
           i += 2
           fact *= i * (i-1)
           num *= x * x
           sign *= -1
           s += num / fact * sign
       getcontext().prec -= 2
       return +s


Decimal 常见问题
================

Q. 总是输入 "decimal.Decimal('1234.5')" 是否过于笨拙。 在使用交互解释
器时有没有最小化输入量的方式？

A. 有些用户会将构造器简写为一个字母：

>>> D = decimal.Decimal
>>> D('1.23') + D('3.45')
Decimal('4.68')

Q. 在带有两个十进制位的定点数应用中，有些输入值具有许多位，需要被舍入
。 另一些数则不应具有多余位，需要验证有效性。 这种情况应该用什么方法？

A. 用 "quantize()" 方法舍入到固定数量的十进制位。 如果设置了 "Inexact"
陷阱，它也适用于验证有效性：

>>> TWOPLACES = Decimal(10) ** -2       # same as Decimal('0.01')

>>> # Round to two places
>>> Decimal('3.214').quantize(TWOPLACES)
Decimal('3.21')

>>> # Validate that a number does not exceed two places
>>> Decimal('3.21').quantize(TWOPLACES, context=Context(traps=[Inexact]))
Decimal('3.21')

>>> Decimal('3.214').quantize(TWOPLACES, context=Context(traps=[Inexact]))
Traceback (most recent call last):
   ...
Inexact: None

Q. 当我使用两个有效位的输入时，我要如何在一个应用中保持有效位不变？

A. 某些运算例如与整数相加、相减和相乘将会自动保留固定的小数位数。 其他
运算，例如相除和非整数相乘则将会改变小数位数，需要再加上 "quantize()"
处理步骤：

>>> a = Decimal('102.72')           # Initial fixed-point values
>>> b = Decimal('3.17')
>>> a + b                           # Addition preserves fixed-point
Decimal('105.89')
>>> a - b
Decimal('99.55')
>>> a * 42                          # So does integer multiplication
Decimal('4314.24')
>>> (a * b).quantize(TWOPLACES)     # Must quantize non-integer multiplication
Decimal('325.62')
>>> (b / a).quantize(TWOPLACES)     # And quantize division
Decimal('0.03')

在开发定点数应用时，更方便的做法是定义处理 "quantize()" 步骤的函数：

>>> def mul(x, y, fp=TWOPLACES):
...     return (x * y).quantize(fp)
>>> def div(x, y, fp=TWOPLACES):
...     return (x / y).quantize(fp)

>>> mul(a, b)                       # Automatically preserve fixed-point
Decimal('325.62')
>>> div(b, a)
Decimal('0.03')

Q. 表示同一个值有许多方式。 数字 "200", "200.000", "2E2" 和 "02E+4" 的
值都相同但有精度不同。 是否有办法将它们转换为一个可识别的规范值？

A. "normalize()" 方法可将所有相同的值映射为统一表示形式：

>>> values = map(Decimal, '200 200.000 2E2 .02E+4'.split())
>>> [v.normalize() for v in values]
[Decimal('2E+2'), Decimal('2E+2'), Decimal('2E+2'), Decimal('2E+2')]

Q. 有些十进制值总是被打印为指数表示形式。 是否有办法得到一个非指数表示
形式？

A. 对于某些值来说，指数表示形式是表示系数中有效位的唯一办法。 例如，将
"5.0E+3" 表示为 "5000" 可以让值保持恒定，但是无法显示原本的两位有效数
字。

如果一个应用不必关心追踪有效位，则可以很容易地移除指数和末尾的零，丢弃
有效位但让值保持不变：

>>> def remove_exponent(d):
...     return d.quantize(Decimal(1)) if d == d.to_integral() else d.normalize()

>>> remove_exponent(Decimal('5E+3'))
Decimal('5000')

Q. 是否有办法将一个普通浮点数转换为 "Decimal"？

A. 是的，任何二进制浮点数都可以精确地表示为 Decimal 值，但完全精确的转
换可能需要比平常感觉更高的精度：

   >>> Decimal(math.pi)
   Decimal('3.141592653589793115997963468544185161590576171875')

Q. 在一个复杂的计算中，我怎样才能保证不会得到由精度不足和舍入异常所导
致的虚假结果。

A. 使用 decimal 模块可以很容易地检测结果。 最好的做法是使用更高的精度
和不同的舍入模式重新进行计算。 明显不同的结果表明存在精度不足、舍入模
式问题、不符合条件的输入或是结果不稳定的算法。

Q. 我发现上下文精度的应用只针对运算结果而不针对输入。在混合使用不同精
度的值时有什么需要注意的吗？

A. 是的。 原则上所有值都会被视为精确值，在这些值上进行的算术运算也是如
此。 只有结果会被舍入。 对于输入来说其好处是“所输入即所得”。 而其缺点
则是如果你忘记了输入没有被舍入，结果看起来可能会很奇怪：

   >>> getcontext().prec = 3
   >>> Decimal('3.104') + Decimal('2.104')
   Decimal('5.21')
   >>> Decimal('3.104') + Decimal('0.000') + Decimal('2.104')
   Decimal('5.20')

解决办法是提高精度或使用单目加法运算对输入执行强制舍入：

   >>> getcontext().prec = 3
   >>> +Decimal('1.23456789')      # unary plus triggers rounding
   Decimal('1.23')

此外，还可以使用 "Context.create_decimal()" 方法在创建输入时执行舍入：

>>> Context(prec=5, rounding=ROUND_DOWN).create_decimal('1.2345678')
Decimal('1.2345')

Q. CPython 实现对于巨大数字是否足够快速？

A. 是的。 在 CPython 和 PyPy3 实现中，decimal 模块的 C/CFFI 版本集成了
高速 libmpdec 库用于实现任意精度正确舍入的十进制浮点算术 [1]。
"libmpdec" 会对中等大小的数字使用 Karatsuba 乘法 而对非常巨大的数字使
用 数字原理变换。

必须要对任意精度算术适配上下文。 "Emin" 和 "Emax" 应当总是设为最大值，
"clamp" 应当总是设为 0 (默认值)。 设置 "prec" 需要十分谨慎。

进行大数字算术的最便捷方式也是使用 "prec" 的最大值 [2]:

   >>> setcontext(Context(prec=MAX_PREC, Emax=MAX_EMAX, Emin=MIN_EMIN))
   >>> x = Decimal(2) ** 256
   >>> x / 128
   Decimal('904625697166532776746648320380374280103671755200316906558262375061821325312')

对于不精确的结果，在 64 位平台上 "MAX_PREC" 的值太大了，可用的内存将会
不足:

   >>> Decimal(1) / 3
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   MemoryError

在具有超量分配的系统上 (即 Linux)，一种更复杂的方式根据可用的 RAM 大小
来调整 "prec"。 假设你有 8GB 的 RAM 并期望同时有 10 个操作数，每个最多
使用 500MB:

   >>> import sys
   >>>
   >>> # Maximum number of digits for a single operand using 500MB in 8-byte words
   >>> # with 19 digits per word (4-byte and 9 digits for the 32-bit build):
   >>> maxdigits = 19 * ((500 * 1024**2) // 8)
   >>>
   >>> # Check that this works:
   >>> c = Context(prec=maxdigits, Emax=MAX_EMAX, Emin=MIN_EMIN)
   >>> c.traps[Inexact] = True
   >>> setcontext(c)
   >>>
   >>> # Fill the available precision with nines:
   >>> x = Decimal(0).logical_invert() * 9
   >>> sys.getsizeof(x)
   524288112
   >>> x + 2
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
     decimal.Inexact: [<class 'decimal.Inexact'>]

总体而言（特别是在没有超量分配的系统上），如果期望所有计算都是精确的则
推荐预估更严格的边界并设置 "Inexact" 陷阱。

[1] 3.3 新版功能.

[2] 在 3.9 版更改: 此方式现在适用于除非整数乘方以外的所有精确结果。 并
    已向下移植到 3.7 和 3.8。

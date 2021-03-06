浮点数对象
**********

PyFloatObject

   这个C类型 "PyObject" 的子类型代表一个Python浮点数对象。

PyTypeObject PyFloat_Type

   这是个属于C类型 "PyTypeObject" 的代表Python浮点类型的实例。在Python
   层面的类型 "float" 是同一个对象。

int PyFloat_Check(PyObject *p)

   当他的参数是一个C类型 "PyFloatObject" 或者是C类型 "PyFloatObject"
   的子类型时，返回真。

int PyFloat_CheckExact(PyObject *p)

   当他的参数是一个C类型 "PyFloatObject" 但不是C类型 "PyFloatObject"
   的子类型时，返回真。

PyObject* PyFloat_FromString(PyObject *str)
    *Return value: New reference.*

   根据字符串 *str* 的值创建一个 "PyFloatObject"，失败时返回 "NULL"。

PyObject* PyFloat_FromDouble(double v)
    *Return value: New reference.*

   根据 *v* 创建一个 "PyFloatObject" 对象，失败时返回 "NULL"。

double PyFloat_AsDouble(PyObject *pyfloat)

   返回一个 C "double" 代表 *pyfloat* 的内容。 如果 *pyfloat* 不是一个
   Python 浮点数对象但是具有 "__float__()" 方法，此方法将首先被调用，
   将 *pyfloat* 转换成一个数点数。 如果 "__float__()" 未定义则将回退至
   "__index__()"。 如果失败，此方法将返回 "-1.0"，因此开发者应当调用
   "PyErr_Occurred()" 来检查错误。

   在 3.8 版更改: 如果可用将使用 "__index__()"。

double PyFloat_AS_DOUBLE(PyObject *pyfloat)

   返回一个 *pyfloat* 内容的 C "double" 表示，但没有错误检查。

PyObject* PyFloat_GetInfo(void)
    *Return value: New reference.*

   返回一个 structseq 实例，其中包含有关 float 的精度、最小值和最大值
   的信息。 它是头文件 "float.h" 的一个简单包装。

double PyFloat_GetMax()

   返回最大可表示的有限浮点数 *DBL_MAX* 为 C "double" 。

double PyFloat_GetMin()

   返回最小可表示归一化正浮点数 *DBL_MIN* 为 C "double" 。

int PyFloat_ClearFreeList()

   清空浮点数释放列表。 返回无法释放的项目数。

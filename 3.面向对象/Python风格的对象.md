# 具有 Python 风格的对象

## 对象表示形式

Python 提供了两种获取对象的字符串表示形式的标准方式

`repr()` 

​	以便于开发者理解的方式返回对象的字符串表示形式

`str()` 

​	以便于用户理解的方式返回对象的字符串表示形式

为了给对象提供其他的表示形式，还会用到另外两个特殊方法：`__bytes__` 和 `__format__`

`bytes()` 函数调用它获取对象的字节序列表示形式。

`__format__` 方法会被内置的 `format()` 函数和 `str.format()` 方法调用，使用特殊的格式代码显示对象的字符串表示形式。

```python
from array import array
import math


class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.x = float(x)
        self.y = float(y)

    def __iter__(self):
        return (i for i in (self.x, self.y))

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (
            bytes([ord(self.typecode)]) + bytes(array(self.typecode, self))) # 迭代 Vector2d 实例，得到一个数组后转为字节序列

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))
```

### 备选构造方法

```python
class Vector2d:
    ...
    
    @classmethod	# 不用传入 self 参数，相反要通过 cls 传入类本身
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)	# 使用 cls 参数构建了一个新实例。
```

### `classmethod` 与 `staticmethod`

`classmethod` 定义操作类，而不是操作实例的方法。`classmethod` 改变了调用方法的方式，因此类方法的第一个参数是类本身，而不是实例。

`classmethod` 最常见的用途是定义备选构造方法，如上面的例子。

`staticmethod` 装饰器也会改变方法的调用方式，但是第一个参数不是特殊的值。其实，静态方法就是普通的函数，只是碰巧在类的定义体中，而不是在模块层定义。

```python
>>> class Demo:
...     @classmethod
...     def klassmeth(*args):
...         return args
...     @staticmethod
...     def statmeth(*args):
...         return args

>>> Demo.klassmeth()
(<class '__main__.Demo'>,)
>>> Demo.klassmeth('spam')
(<class '__main__.Demo'>, 'spam')

>>> Demo.statmeth()
()
>>> Demo.statmeth('spam')
('spam',)
```

### 格式化显示

内置的 `format()` 函数和 `str.format()` 方法把各个类型的格式化方式委托给相应的 `.__format__(format_spec)` 方法。`format_spec` 是格式说明符，它是：

- `format(my_obj, format_spec)` 的第二个参数
- `str.format()` 方法的格式字符串，`{}` 里代换字段中冒号后面的部分

```python
>>> brl = 1/2.43

>>> brl
0.4115226337448559

>>> format(brl, '0.4f')
'0.4115'

>>> '1 BRL = {rate:0.2f} USD'.format(rate=brl)
'1 BRL = 0.41 USD'
```

格式规范微语言为一些内置类型提供了专用的表示代码。比如，`b` 和 `x` 分别表示二进制和十六进制的 `int` 类型，`f` 表示小数形式的 `float` 类，而 `%` 表示百分数形式

```python
>>> format(42, 'b')
'101010'
>>> format(2/3, '.1%')
'66.7%'
```

格式规范微语言是可扩展的，因为各个类可以自行决定如何解释 `format_spec` 参数。例如，`datetime` 模块的类，它们的 `__format__` 方法使用的格式代码与 `strftime()` 函数一样。

```python
>>> from datetime import datetime

>>> now = datetime.now()
>>> format(now, '%H:%M:%S')
'16:22:39'
>>> "It`s now {:%I:%M %p}".format(now)
'It`s now 04:22 PM'
```

如果类没有定义 `__format__` 方法，从 `object` 继承的方法会返回 `str(my_object)` 

```python
>>> v1 = Vector2d(3, 4)

>>> format(v1)
'(3.0, 4.0)'
```

```python
# 给 Vector2d 类中定义
def __format__(self, fmt_spec=''):
    components = (format(c, fmt_spec) for c in self)
    return '({}, {})'.format(*components)

def angle(self):
	return math.atan2(self.y, self.x)
->
def __format__(self, fmt_spec=''):
    if fmt_spec.endswith('p'):
        fmt_spec = fmt_spec[:-1]
        coords = (abs(self), self.angle())
        outer_fmt = '<{}, {}>'
    else:
        coords = self
        outer_fmt = '({}, {})'
    components = (format(c, fmt_spec) for c in self)
    return outer_fmt.format(*components)
```

### 散列之

为了把 `Vector2d` 实例变成可散列的，必须使用 `__hash__` 方法（还需 `__eq__` 方法）。此外，还要让向量不可变。

让 `Vector2d` 不可变

```python
class Vector2d:
	typecode = 'd'
    
    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)
    
    @property
    def x(self):
        return self.__x
    
    @property
    def y(self):
        return self.__y
    
    def __iter__(self):
        return (i for i in (self.x, self.y))
    
    def __hash__(self):
        return hash(self.__x) ^ hash(self.__y)	# ^ 异或
```

## Python 的私有属性和“受保护的”属性

Python 不能像 Java 那样使用 private 修饰符创建私有属性，但是 Python 有个简单的机制，能避免子类意外覆盖“私有”属性。

使用 `__xx` 的形式命名实例属性，Python 会把属性存入实例的 `__dict__` 属性中，而且会在前面加上一个下划线和类名。比如对于 `Dog` 类来说，`__mood` 会变成 `_Dog__mood`；`Beagele` 类来说，会变成 `_Beagle__mood` 。这个语言特性叫名称改写（name mangling）。

但是这样子的方式有点烦人。

现在大部分约定使用一个下划线前缀编写“受保护”的属性（如 `self._x`）。他们不会在类外部访问这些属性。

## 使用 `__slots__` 类属性节省空间

默认情况下，Python 在各个实例中名为 `__dict__` 的字典里存储实例属性。为了使用底层的散列表提升访问速度，字典会消耗大量内存。如果要处理数百万属性不多的实例，通过 `__slots__` 类属性可以节省大量内存，方法是让解释器在元组中存储实例属性，而不用字典。

定义 `__slots__`  的方式是，创建一个类属性，使用 `__slots__` 这个名字，并把它的值设为一个字符串构成的可迭代对象，其中各个元素表示各个实例属性。

```python
class Vector2d:
	__slots__ = ('__x', '__y')
    
    typecode = 'd'
    ...
# 在类定义 `__slots__` 属性之后，实例不能再有 `__slots__` 中所列名称之外的其他属性。
```

注意点：

- 不能滥用，不用使用它限制用户能赋值的属性。
- 处理列表数据时 `__slots__` 属性最有用，例如模式固定的数据库记录，以及特大型数据集。
- 每个子类都要定义 `__slots__` 属性，因为解释器会忽略继承的 `__slots__` 属性。
- 实例只能拥有 `__slots__` 列出来的属性，除非把 `__dict__` 加进去。（失去节省内存的功效）。
- 如果不把 `__weakref__` 加入 `__slots__`，实例就不能作为弱引用的目标。

## 覆盖类属性

Python 有个独特的特性：类属性可用于为实例属性提供默认值。

```python
>>> v1 = Vector2d(1.1, 2.2)
>>> dumpd = bytes(v1)
>>> dumpd
b'd\x9a.....\x01@'
>>> len(dumpd)
17
>>> v1.typecode = 'f'
>>> dumpf = bytes(v1)
>>> dumpf
b'f\xcd....\x0c@'
>>> len(dumpf)
9
>>> Vector2d.typecode
'd'
# 如果要修改类属性的值
>>> Vector2d.typecode = 'f'
```

更符合 Python 风格并效果更持久的方式

```python
>>> class ShortVector2d(Vector2d):
    	typecode = 'f'
>>> sv = ShortVector2d(1/11, 1/27)
# 定义子类，覆盖 typecode 类属性
```


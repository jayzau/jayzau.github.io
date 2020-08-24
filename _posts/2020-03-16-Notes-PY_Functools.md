---
layout: post
title: Python内置包Functools学习
date: 2020-03-16
tags: 备忘录   
---

# Functools

[functools](https://docs.python.org/3.7/library/functools.html)模块用于高阶函数：作用于或返回其他函数的函数。一般来说，任何可调用对象都可以作为函数来处理。

Python Vsersion 3.7

### `functools.cmp_to_key(func)`

将旧式比较函数（old-style comparison function）转换为关键字函数（key function）。使用接受关键字函数的工具（如[`sorted()`](https://docs.python.org/3/library/functions.html#sorted)，[`min()`](https://docs.python.org/3/library/functions.html#min)， [`max()`](https://docs.python.org/3/library/functions.html#max)，[`heapq.nlargest()`](https://docs.python.org/3/library/heapq.html#heapq.nlargest)，[`heapq.nsmallest()`](https://docs.python.org/3/library/heapq.html#heapq.nsmallest)， [`itertools.groupby()`](https://docs.python.org/3/library/itertools.html#itertools.groupby)）。此函数就是个转换工具，主要用作将程序转换为不再支持比较函数的Python 3。

比较函数任何时候在调用时会接收两个参数用于比较，返回负数表示小于，零表示等于，正数表示大于。一个关键字函数调用时会接收一个参数，然后返回一个值用来排序。

**示例：**

```python
sorted(iterable, key=cmp_to_key(locale.stroll))
```

**实例：**

```python
from functools import cmp_to_key


class Student:
    def __init__(self, name, grade, age):
        self.name = name
        self.grade = grade
        self.age = age

    def __repr__(self):
        return repr((self.name, self.grade, self.age))


student_objects = [
    Student('john', 'A', 15),
    Student('jane', 'B', 12),
    Student('dave', 'B', 10),
]


def compares(a: Student, b: Student):
    if a.grade > b.grade:       # 自定义规则1
        return 1
    elif a.grade < b.grade:
        return -1
    if a.age > b.age:       # 自定义规则2
        return 1
    elif a.age < b.age:
        return -1
    return 0


r1 = sorted(student_objects, key=lambda student: (student.grade, not student.age))      # 自定义规则
r2 = sorted(student_objects, key=cmp_to_key(compares))

print(r1)
# [('john', 'A', 15), ('jane', 'B', 12), ('dave', 'B', 10)]
print(r2)
# [('john', 'A', 15), ('dave', 'B', 10), ('jane', 'B', 12)]
```

---

### `functools.total_ordering(cls)`

给一个类定义一个或多个比较排序方法，然后这个类装饰器补充其余的排序方法。

这个类必须定义 [`__lt__()`](https://docs.python.org/2/reference/datamodel.html#object.__lt__) ，[`__le__()`](https://docs.python.org/2/reference/datamodel.html#object.__le__)，[`__gt__()`](https://docs.python.org/2/reference/datamodel.html#object.__gt__) ，[`__ge__()`](https://docs.python.org/2/reference/datamodel.html#object.__ge__)中的一种，除此之外，还必须实现 [`__eq__()`](https://docs.python.org/2/reference/datamodel.html#object.__eq__) 方法。

**示例：**

```python
@total_ordering
class Student:
    def __eq__(self, other):
        return ((self.name.lower(), self.grade.lower()) ==
                (other.name.lower(), other.grade.lower()))
    def __lt__(self, other):
        return ((self.name.lower(), self.grade.lower()) <
                (other.name.lower(), other.grade.lower()))
```

**实例：**

```python
# 实例使用的上面定义的Student类
s1 = Student('john', 'A', 15)
s2 = Student('jane', 'B', 12)
s3 = Student('dave', 'B', 10)

print(s1 >= s2)
print(s2 < s1)
print(s3 == s1)

# 使用装饰器结果
"""
True
True
False
"""
# 不使用装饰器结果
"""
TypeError: '>=' not supported between instances of 'Student' and 'Student'
"""
# 另外，不定义__eq__，也只是无法比较，并没有报错
```

---

### `functools.reduce(function, iterable[, initializer])`

和函数[`reduce()`](https://docs.python.org/2/library/functions.html#reduce)一样，向前兼容Python3的。

Python2:`import reduce`

Python3:`from functools import reduce`

**实例：**

```python
reduce(lambda x, y: x + y, [1, 2, 3, 4, 5])
# (((1 + 2) + 3) + 4) + 5)
# 15
reduce(lambda x, y: x if x > y else y, [1, 4, 7, 3, 9, 2, 1, 3])
# 9
```

---

### `functools.partial(func[,*args][, **keywords])`

给一个函数传参，但不会直接调用，而是返回一个新的 [partial 对象](https://docs.python.org/2/library/functools.html#partial-objects) ，这个对象可以继续追加参数。

如果追加的是位置参数（args），会依次传入；

如果追加的是关键字参数（kwargs），会覆盖原有参数；

类似这样：

```python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)	  # 关键字参数追加
        return func(*(args + fargs), **newkeywords)		# 位置参数追加
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

**示例：**

```python
>>> from functools import partial
>>> basetwo = partial(int, base=2)
>>> basetwo.__doc__ = 'Convert base 2 string to an int.'
>>> basetwo('10010')
18
```

**实例：**

```python
from functools import partial


def add(*args, **kwargs):
    return sum(args) + sum(kwargs.values())


obj = partial(add, 1, 2)	# add(1, 2)
print(obj(3))	# add(1, 2, 3) => 6
print(obj(3, 4))	# add(1, 2, 3, 4) => 10

obj2 = partial(add, a=1, b=2)	# add(a=1, b=2)
print(obj2(c=5))	# add(a=1, b=2, c=5) => 8
print(obj2(b=7, c=5))	 # add(a=1, b=7, c=5) => 13

# 参数对应位置顺序
def example(a, *args, kw=None, **kwargs):
    pass
```

---

### `functools.partialmethod(func, *args, **keywords)`

和`partial`类似，示例是放在类里面使用的，可以将`self`作为默认值传进去。

**示例：**

```python
from functools import partialmethod


class Cell(object):
    def __init__(self):
        self._alive = False

    @property
    def alive(self):
        return self._alive

    def set_state(self, state):
        self._alive = bool(state)

    set_alive = partialmethod(set_state)
    set_dead = partialmethod(set_state, False)


c = Cell()
print(c.alive)		# False
c.set_alive(True)
print(c.alive)		# True
```

> Python Version >= 3.4

---

### `functools.singledispatch`

有点厉害，将函数转换成一个单一调用通用函数。使用`@singledispatch`来装饰函数。

比如说业务需要转换一种数据，数据类型不确定，但是只有一个入口。那么不同类型的数据肯定得分方法处理。接下来用三种不同写法展示，就一目了然了。

- **初级写法**

  ```python
  def run(value):
      if type(value) == int:
          print("处理整数")
      elif type(value) == float:
      	print("处理浮点数")
      elif type(value) == str:
          print("处理字符串")
      else:
          print("处理不了")
  ```

- **中级写法**

  ```python
  def run(value):
      if isinstance(value, int):
          print("处理整数")
      elif isinstance(value, float):
      	print("处理浮点数")
      elif isinstance(value, str):
          print("处理字符串")
      else:
          print("处理不了")
  ```

- **高级写法**

  ```python
  from functools import singledispatch
  
  
  @singledispatch
  def run(value):
      print("参数类型不确定，无法处理", value, type(value))
  
  
  @run.register
  def _(value: int):		# 类型指定方式1
      print("整数类型在这里处理", value, type(value))
  
  
  @run.register
  def _(value: float):
      print("浮点数类型在这里处理", value, type(value))
  
  
  @run.register(str)		# 类型制定方式2
  def _(value):
      print("字符串类型在这里处理", value, type(value))
  
  
  @run.register(dict)
  @run.register(list)		# 可以多个类型叠加
  def _(value):
      print("字典和列表通吃", value, type(value))
  
  
  def type_bool(value):
      print("布尔类型在这里处理", value, type(value))
  
  
  run.register(bool, type_bool)	 # 可以事后再指定类型
  # ------------------------------------------------
  
  run(1)
  run(1.0)
  run("1")
  run(True)
  run(None)
  run.dispatch(str)({"1": 1})		# 强行指定对应类型函数处理
  run([1])
  # 结果
  """
  整数类型在这里处理 1 <class 'int'>
  浮点数类型在这里处理 1.0 <class 'float'>
  字符串类型在这里处理 1 <class 'str'>
  布尔类型在这里处理 True <class 'bool'>
  参数类型，不确定 None <class 'NoneType'>
  字符串类型在这里处理 {'1': 1} <class 'dict'>
  字典和列表通吃 [1] <class 'list'>
  """
  ```

很明显，用`singledispatch`实现的代码，更加灵活、美观。

> Python Version >= 3.4

---

### `functools.update_wrapper(wrapper, wrapped[, assigned][, updated])`

保证函数原有属性不会变化，

> ```python
> WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',
>                        '__annotations__')
> WRAPPER_UPDATES = ('__dict__',)
> ```

并且添加原有未设置属性。

**实例：**

```python

def wrapper(f):
    def wrapper_function(*args, **kwargs):
        """这个是修饰函数"""
        return f(*args, **kwargs)
    # update_wrapper(wrapper_function, f)
    return wrapper_function


# @wrapper
def wrapped():
    """这个是被修饰的函数"""
    pass


print("__module__:", wrapped.__module__)
print("__name__:", wrapped.__name__)
print("__qualname__:", wrapped.__qualname__)
print("__doc__:", wrapped.__doc__)
print("__annotations__:", wrapped.__annotations__)
print("__dict__:", wrapped.__dict__)


"""     # 不使用 wrapper 装饰器装饰 
__module__: __main__
__name__: wrapped
__qualname__: wrapped
__doc__: 这个是被修饰的函数
__annotations__: {}
__dict__: {}
"""
"""     # 使用 wrapper 装饰器装饰， 不使用 update_wrapper
__module__: __main__
__name__: wrapper_function
__qualname__: wrapper.<locals>.wrapper_function
__doc__: 这个是修饰函数
__annotations__: {}
__dict__: {}
"""
"""     # 使用 wrapper 装饰器装饰， 也使用 update_wrapper
__module__: __main__
__name__: wrapped
__qualname__: wrapped
__doc__: 这个是被修饰的函数
__annotations__: {}
__dict__: {'__wrapped__': <function wrapped at 0x7f8a4ba14830>}
"""
```

---

### `functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)`

在`update_wrapper`的基础上多包了一层，源代码如下：

```python
WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',
                       '__annotations__')
WRAPPER_UPDATES = ('__dict__',)

def wraps(wrapped,
          assigned = WRAPPER_ASSIGNMENTS,
          updated = WRAPPER_UPDATES):
    """Decorator factory to apply update_wrapper() to a wrapper function

       Returns a decorator that invokes update_wrapper() with the decorated
       function as the wrapper argument and the arguments to wraps() as the
       remaining arguments. Default arguments are as for update_wrapper().
       This is a convenience function to simplify applying partial() to
       update_wrapper().
    """
    return partial(update_wrapper, wrapped=wrapped,
                   assigned=assigned, updated=updated)
```

所以重点是理解`update_wrapper`就OK。
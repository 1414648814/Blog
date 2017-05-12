# 前言

这篇博客是在学习某个网站时记录下来的，所以其纪录的顺序和那个网站里面一样，有些知识点已经大概了解了就不再赘述。

# 基础

## 字符串和编码

在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码。
在 `python 3` 版本中，字符串是以Unicode来编码的；当你的源代码包含中文的时候，需要指定保存为UTF-8编码：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

## 使用list和tuple

### list

函数 | 描述
---|---
len (list) | 获取元素个数
cmp (list1, list2) | 比较两个列表的元素
max (list) | 列表元素最大值
min(list) | 列表元素最小值
list (seq) | 将元祖转换为列表（()->[]）
append (obj) | 追加元素到末尾
count (obj) | 统计某个元素在列表中出现的次数
extend (seq) | 在原来末尾追加另一个序列
index (obj) | 找到第一个匹配项的索引位置
insert (index, obj) | 在指定位置加入
pop (index) | 移除指定索引的元素（不填则默认最后一个），并且返回
remove (obj) | 移除第一个匹配项
reverse () | 反转
sort ([func]) | 排序

### tuple

和list类似，但是tuple在初始化之后则不能修改（指向的元素的地址不变）。如果可能，用tuple取代list，因为更安全一些；

注意：

在定义只有一个元素的tuple时，在元素末尾也会加上一个 `,`

## 使用dict和set

### dict

函数 | 描述
---|---
in | 判断key是否存在
get (obj) | 同上，同时如果不存在，还可以返回自定义的值
pop (obj) | 删除key
clear() | 清空
items() | 返回可以遍历的健值对列表
keys() | 以列表形式返回字典中所有的键
values() | 以列表返回字典中的所有值

### set

同样是一组key的集合（key不重复），但是不存储value。

函数 | 描述
---|---
add (key) | 添加key
remove (key) | 删除key

# 高级特性

## 切片

如果想取数组中的部分元素，则可以通过在数组中使用 `[start:end]` 实现，类似于JavaScript中的 `slice(start, end)`；其中`start和end`都可以为负数，表示以倒数的方式来计算，也可以只写一个负数，也可以什么都不写只有一个：，表示复制整个数组。

如果是`::`，表明每隔一段来取一个，比如：

```python
>>> 'ABCDEFG'[::2]
'ACEG'

```

也可以这么写，前6个数每2个取1个：

```python
>>> 'ABCDEFG'[:6:2]
'BDF'

```

切片适用于类似数组的类型，比如 字符串，list，tuple等等。

## 迭代

在python中，迭代使用的是`for...in...`关键字，同时只要是`可迭代对象`，都可以迭代，比如json对象等等。

那么，如何判断一个对象是可迭代对象呢？方法是通过collections模块的Iterable类型判断：

```python
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False

```

## 列表生成式

将所有的表达式放到`[]`里面去执行：

```python
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]

```

## 生成器

怎么创建一个生成器呢？

1. 将`列表生成式`中外围的`[]`改为`()`即可

    可以通过`next`函数获取`generator`的下一个返回值。
    
    ```python
    >>> L = [x * x for x in range(10)]
    >>> L
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
    >>> g = (x * x for x in range(10))
    >>> g
    <generator object <genexpr> at 0x1022ef630>
    
    >>> next(g)
    0
    >>> next(g)
    1
    
    ```

2. 在函数中使用`yield`关键字，其作用类似于return＋print

    ```python
    def odd():
        print('step 1')
        yield 1
        print('step 2')
        yield(3)
        print('step 3')
        yield(5)
    
    ```
    
    在执行过next后，再次执行的时候则从上一次执行`yield`后的位置继续执行，遇见`yield`则返回，但是如果执行到后面已经没有`yield`后，则会报错。
    
    ```python
    >>> o = odd()
    >>> next(o)
    step 1
    1
    >>> next(o)
    step 2
    3
    >>> next(o)
    step 3
    5
    ```
    
    如果不想用`next`，一步一步的计算，同样的可以使用循环。
    
    ```python
    def fib(max):
        n, a, b = 0, 0, 1
        while n < max:
            yield b
            a, b = b, a + b
            n = n + 1
        return 'done'
        
    >>> for n in fib(6):
    ...     print(n)
    ...
    1
    1
    2
    3
    5
    8
    
    ```
    
    如果想要捕捉到返回值，则在找不到next的时候（异常）返回，例如：
    
    ```python
    >>> g = fib(6)
    >>> while True:
    ...     try:
    ...         x = next(g)
    ...         print('g:', x)
    ...     except StopIteration as e:
    ...         print('Generator return value:', e.value)
    ...         break
    ...
    g: 1
    g: 1
    g: 2
    g: 3
    g: 5
    g: 8
    Generator return value: done
    
    ```
    
# 迭代器

注意区分 `Iterable和Iterator`，前者是可迭代的，后者是迭代器（类似于C++中vector和::iterator），同样的，可以使用`isinstance`来判断；


# 函数式编程

## 高阶函数

### map/reduce

`map(函数，Iterable)`，将可迭代对象中的元素执行函数以后生成一个新的iterator；

```python
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]

```

`reduce函数`必须接收两个参数，一个是函数，一个是list，其重点在于将list中的元素进行函数计算后作用到下一个元素上。

```python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)

```

比如这么用：

```python
from functools import reduce

def str2int(s):
    def fn(x, y):
        return x * 10 + y
    def char2num(s):
        return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
    return reduce(fn, map(char2num, s))
    
```

### filter

用于过滤序列，返回一个新的序列；

和`map`一样，同样接收一个函数和序列，但不同的是，`filter`使用函数对序列中的元素进行筛选，也就是说该函数是一个`bool`函数，将值传递进去后只有返回成功才将该值加入到新的队列中。

```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]

```

> 如果这不是一个筛选函数呢？python则认为均为false，也就是不加入任何元素，返回空序列。


### sorted

对序列进行排序，后面还可以加上key来指定排序函数，还可以加上reverse表示反向排序；

```python
class Student:
        def __init__(self, name, grade, age):
                self.name = name
                self.grade = grade
                self.age = age
        def __repr__(self):
                return repr((self.name, self.grade, self.age))

# student_objects = [
#        Student('john', 'A', 15),
#        Student('jane', 'B', 12),
#        Student('dave', 'B', 10),
#]

# 使用lambda
sorted(student_objects, key=lambda student: student.age, reverse=True)   
# sort by age
# [('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]

# 或者使用attrgetter
sorted(student_objects, key=attrgetter('age'), reverse=True)

```

## 闭包

当你调用了一个函数A，这个函数A返回内部的函数B，这个返回的函数B就是闭包；

```python
def func(name):
    def inner_func(age):
        print 'name:', name, 'age:', age
    return inner_func

bb = func('the5fire')
bb(26)  # >>> name: the5fire age: 26

```

## 匿名函数

lambda表示匿名函数（和C++ 11.0中的lambda表达式类似），格式为：

```python
lambda x : x * x

```

冒号前面表示参数，返回值为冒号后面的计算结果。


## 装饰器

**需要单独写一篇文章进行总结（重要）**

总之就是在不改变一个函数的前提下（其它调用该函数的也不需要改变，只需要在调用函数前面加上`@decorator`就行），往该函数上加功能进行任意的扩展。


## 偏函数

`functools.partial`表示的是将函数“保存”下来，返回来一个在原来函数的基础上重新定义的函数；

例子：

```python
import functools

def add(a, b):
    return a + b

add(4, 2)
6

plus3 = functools.partial(add, 3)
plus5 = functools.partial(add, 5)

plus3(4)
7

plus5(10)
15

```

除此之外，`functools`模块还包括：

* **functool.update_wrapper**：从原始对象拷贝或加入现有partial对象
* **functool.wraps**：调用函数装饰器partial
* **functools.reduce**：等同于内置函数reduce()
* **functools.cmp\_to\_key**：将老式鼻尖函数转换成key函数，用在接受key函数的方法中
* **functools.total\_ordering**：它是针对某个类如果定义了\_\_lt\_\_、le、gt、\_\_ge\_\_这些方法中的至少一个，使用该装饰器，则会自动的把其他几个比较函数也实现在该类中。


# 面向对象编程

## 继承和多态

对于静态语言（例如Java）来说，如果需要传入Animal类型，则传入的对象必须是Animal类型或者它的子类，否则，将无法调用run()方法。

对于Python这样的动态语言来说，则不一定需要传入Animal类型。我们只需要保证传入的对象有一个run()方法就可以了：

## 实例属性和类属性

> 在编写程序的时候，千万不要把实例属性和类属性使用相同的名字，因为相同名称的实例属性将屏蔽掉类属性，但是当你删除实例属性后，再使用相同的名称，访问到的将是类属性，个人不建议在示例上添加任何新的属性，在类中加上属性即可。

# 面向对象高级编程

## 使用__slots__

因为python是一门动态语言，所以无法避免的会有给一个实例添加各种的属性或者方法的情况，python可以使用这个来限制实力的属性。

```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

```

但是我觉得这样也不是很好，如果有很多属性的话岂不是都写一长串，而且如果有人忘写了，还是不可避免的出现之前那种说过的情况。总而言之，语言不会去限制你怎么写，一切靠自觉。


## @property

看到这个的第一感觉就是类似`Object-C`里面的property；

只需要在属性对应的`get函数`前面加上`@property`，则可以实现get方法；在`set函数`前面加上`属性.setter`，则可以实现set方法；


## 多继承

在类定义的时候，可以在括号里面加上多个类从而实现多继承；

```python
class Dog(Animal,Friend) //狗是一种动物也是一种朋友

```

## 运算符重载

运算符重载意味着在类方法中拦截内置的操作－当类的实例出现在内置操作中，python自然会调用你的方法，比如我们一直使用的`__init__`；

> 类可以重载所有Python表达式运算符，像是打印、函数调用、属性点号运算等内置运算

常见运算符重载方法包括以下：

![](http://odwv9d2u8.bkt.clouddn.com/17-4-22/91487689-file_1492821363959_24cf.png)


## 枚举类

像是在`c++`中的`enum`，使用方法如下：

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

```

## 元类

元类就是用来创建类的东西，包括：

1. 拦截类的创建；
2. 修改类；
3. 返回修改后的类；

个人觉得这是一个黑魔法，一般项目中应该不会用到，同时使用者要很小心，而且要很清楚自己在干嘛。

### type

`type`函数除了可以帮我们获取对象的类型，还可以帮我们创建对象，其描述为：

```python
type(类名, 父类的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)

```

举个例子：

```python
# 我们可以这么做

class MyClass(object):
    bar = True
    
# 也可以这么做
Foo = type('Foo', (), {'bar':True})

```

上面两种方法达到的目的是一样的，都能够创建一个类，但正常情况下不建议这么使用；

### __metaclass__

除了使用`type`可以动态的创建类意外，还可以使用`metaclass`；

```python
class Foo(object):
	__metaclass__ = dosomething

```

如果这么做的话，python就不会使用内建的type来创建这个类，而是使用你自己定义的元类来创建，python会做这么一个操作：

> 首先写下class Foo(object)，但是类对象Foo还没有在内存中创建。Python会在类的定义中寻找__metaclass__属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。


例子：

```python
# 请记住，'type'实际上是一个类，就像'str'和'int'一样
# 所以，你可以从type继承
class UpperAttrMetaClass(type):
    # __new__ 是在__init__之前被调用的特殊方法
    # __new__是用来创建对象并返回之的方法
    # 而__init__只是用来将传入的参数初始化给对象
    # 你很少用到__new__，除非你希望能够控制对象的创建
    # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
    # 如果你希望的话，你也可以在__init__中做些事情
    # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
    def __new__(upperattr_metaclass, future_class_name, future_class_parents, future_class_attr):
        attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
        return super(UpperAttrMetaClass, upperattr_metaclass).__new__(upperattr_metaclass, future_class_name, future_class_parents, uppercase_attr)


class Foo(object):
    bar = "happy"

    def echo_bar(self):
        print(self.bar)

myclass = UpperAttrMetaClass('FooChild', (Foo,), {})

my = myclass()
my.echo_bar()  #输出happy

```

# 错误、调试和测试

## 错误处理

类似于c++中的try...catch，python也有类似的错误处理函数：
 
```python
try...
except: division by zero
finally...
```
 
例子：

```python
try:
    print('try...')
    r = 10 / int('a')
    print('result:', r)
except ValueError as e:
    print('ValueError:', e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
finally:
    print('finally...')
print('END')

```

你还可以使用内置的logging来纪录错误信息：

```python
import logging
def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

```


# 总结

到这里，我觉得最基本的语法已经学完了，接下来的包括：

* IO编程
* 异步编程
* 进程和线程编程
* 网络编程
* 数据库编程

这些编程都是用到的时候再来查询总结。
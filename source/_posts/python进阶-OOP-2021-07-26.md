---
title: python进阶-OOP
date: 2021-07-26 11:39:13
categories:
  - - 机器学习
  - - 计算机语言
  - - 研究生课程基础
  - - Python
tags:
  - 机器学习
  - python
  - 面向对象
  - OOP
  - 进阶
mathjax: true
headimg: 'https://z3.ax1x.com/2021/08/05/fegc38.md.png'
---

python学习笔记，面对对象部分

<!-- more -->

## OOP

作为长期使用C++，java进行开发的程序员老说，OOP可以说是一种比较情切的程序设计思想了，所谓万物皆对象。

在python，几乎所有数据类型都可视为对象，甚至函数，python同样支持自定义对象。

### 模块

---

在了解python中实现OOP之前，先来描述一下模块的概念，如下是一个自定义模块的样子：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'Ender'

# 以上是模块的文档规范

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

第10行中，在导入`sys`模块后，变量`sys`就指向了该模块，此后便可通过该变量访问这个模块中的功能。

例如第13行的`argv`变量，就是该模块中用于存储命令行中参数的`list`，`argv`中至少包含一个元素，即`.py`文件的名称。

例如使用如下命令调用时：

```powershell
python3 hello.py Ender
```

`argv`中的参数就是：

```python
['hello.py', 'Michael']
```

当我们在命令行运行`hello`模块文件时，python解释器把一个特殊的变量`__name__`置为`__main__`，而如果在其他地方导入该`hello`模块时，`if`判断将失败。因此，这种`if`测试可以让一个模块通过命令行运行时执行一些额外的代码，最常见的用途就是运行测试。

例如使用命令行运行`hello.py`

```powershell
>>> python3 hello.py
Hello, world!
>>> python hello.py Ender
Hello, Ender!
```

而导入`hello`模块时，不会打印任何东西，原因是没有调用`test`函数：

```powershell
>>> python
>>> import hello
>>> hello.text()
>>> Hello, world!
```

#### 模块内作用域

---

Python中没有严格的作用域限定方式，只能依靠某些特定的命名方式以及约定俗称来限定模块间对象的访问规则：

1. 能够被外部访问的模块内对象只能以**字母**开头
2. 特殊对象，如声明作者，调用判断，文档说明等，均以`__`开头以及结尾，如`__author__`, `__name__`, `__doc__`
3. 非公开变量需要使用下划线开头，例如`_xxx`, `__xxx`，这样的对象不应该被直接引用。

### 类

---

接下来我们看一些python中的相关操作。

Python中定义类通过`class`关键字

```python
class Student(object):
    pass
```

`class`后面紧接着是类名，即`Student`，类名通常是大写开头的单词，紧接着是`(object)`，表示该类是从哪个类继承下来的，继承的概念我们后面再讲，通常，如果没有合适的继承类，就使用`object`类，这是所有类最终都会继承的类。

实例化则通过如下方式：

```powershell
>>> bart = Student()
>>> bart
<__main__.Student object at 0x10a67a590>
>>> Student
<class '__main__.Student'>
```

创建实例后，可以为某一单独实例绑定属性：

```powershell
>>> bart.name = 'Bart Simpson'
>>> bart.name
'Bart Simpson'
```

而为类绑定属性，则需要使用一个特殊的方法：`__init__`：

```python
class Student(object):
    def __init__(self, name, score):
        self.name = name
        self.score = score
```

其中第一个参数`self`表示创建的实例本身，因此在该函数内会将各种属性绑定到`self`即`self`所指的实例。

```python
>>> bart = Student('Bart Simpson', 59)
>>> bart.name
'Bart Simpson'
>>> bart.score
59
```

可见实例本身不用显式的传入实例。

为类创建方法：

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
```

调用时直接使用：

```powershell
>>> bart.print_score()
Bart Simpson: 59
```

可见`self`也不需要显示的传入

### 访问限制

---

有时为了更好的封装某个类，我们通常会对类中的变量进行访问权限的限制。在python中，没有特定的访问限制关键字，而是通过特殊的变量名进行限制。

如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线`__`，在Python中，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问，例如：

```python
class Student(object):

    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))
```

再次访问将得到如下结果：

```powershell
>>> bart = Student('Bart Simpson', 59)
>>> bart.__name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute '__name'
```

此时可以通过为私有变量增加get和set方法创建接口来访问他们。

还需要注意的是，在Python中，变量名类似`__xxx__`的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是private变量，所以，不能用`__name__`、`__score__`这样的变量名。

而有些时候，你会看到以一个下划线开头的实例变量名，比如`_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。

而模块中讲到，python中作用域限定通常依靠约定俗称。其实是类中的访问权限限制也是如此。

不能直接访问`__name`是因为Python解释器对外把`__name`变量改成了`_Student__name`，所以，仍然可以通过`_Student__name`来访问`__name`变量：

```python
>>> bart._Student__name
'Bart Simpson'
```

但不同版本的Python解释器可能会把`__name`改成不同的变量名。

另外，在修改操作上，Python的访问限制也没有达到理想效果：

```powershell
>>> bart = Student('Bart Simpson', 59)
>>> bart.get_name()
'Bart Simpson'
>>> bart.__name = 'New Name' # 设置__name变量！
>>> bart.__name
'New Name'
```

表面上看，外部代码“成功”地设置了`__name`变量，但实际上这个`__name`变量和class内部的`__name`变量*不是*一个变量！内部的`__name`变量已经被Python解释器自动改成了`_Student__name`，而外部代码给`bart`新增了一个`__name`变量：

```python
>>> bart.get_name() # get_name()内部返回self.__name
'Bart Simpson'
```

### 继承与多态

---

在类的章节有讲到python中继承的方式：

```python
Class Animal(object):
    def run(self):
        print('Animal is running...')
    def Introduce(self):
        print('I\'m an animal')
```

如上类就是继承了object类的一个Animal类，我们可以继续编写它的子类：

```python
class Dog(Animal):
    def run(self):
        print('Dog is running...')
    
    def eat(self):
        print('Eating meat...')

class Cat(Animal):
    def run(self):
        print('Cat is running...')
    
    def eat(self):
        print('Eating Fish...')
```

可见Dog和Cat类继承了Animal的方法，并且在其中重写了`run`方法

Python中创建一个类，实际上是定义了一种新的数据类型，也就意味着：

```python
a = list()
b = Animal()
c = Dog()

isinstance(a, list)  # True
isinstance(b, Animal)  # True
isinstance(c, Dog)  # True
isinstance(c, Animal)  # True
```

可见`c`即是`Dog`也是`Animal`，那么如下操作也是合法的：

```python
def running(animal):
    animal.run()


running(Animal())  # Animal is running
running(Dog())  # Dog is running
running(Cat())  # Cat is running
```

可见`Dog`与`Cat`的实例能够被接受且均能调用`run()`方法。

这样的特性能够让我们很方便的实现**“开闭”原则**即：

1. 对扩展开放：允许新增`Animal`子类
2. 对修改封闭：不需要修改以来`Animal`类型的`running`等函数

#### 静态语言与动态语言

---

上面的例子中的`running`函数看起来是用了一个`Animal`类型的变量接受了参数，但由于Python是动态语言，实际上传入`running`函数前，编辑器并不知道将传入的是个怎样的类型，这就意味着，只要我们传入的参数包含`run`方法，那么代码就能正常运行，例如：

```python
class Timer(object):
    def run(self):
        print('Start...')

running(Timer())  # 'Start...'
```

而在Java和C++中，这样的参数传入是不被允许的。

这就是动态语言中的`鸭子类型`，它并不要求严格的继承体系，所谓*一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。*

Python的“file-like object“就是一种鸭子类型。对真正的文件对象，它有一个`read()`方法，返回其内容。但是，许多对象，只要有`read()`方法，都被视为“file-like object“。许多函数接收的参数就是“file-like object“，你不一定要传入真正的文件对象，完全可以传入任何实现了`read()`方法的对象。

### 获取对象信息

---

当我们拿到一个实例对象时，有两种方法可以确定他们的类：

1. `type()`
2. `instance()`

#### type

---

```powershell
>>> type(123)
<class 'int'>
>>> type('str')
<class 'str'>
>>> type(None)
<type(None) 'NoneType'>
>>> type(abs)
<class 'builtin_function_or_method'>
>>> type(a)
<class '__main__.Animal'>
>>> type(123)==type(456)
True
>>> type(123)==int
True
>>> type('abc')==type('123')
True
>>> type('abc')==str
True
>>> type('abc')==type(123)
False
```

如果需要判断一个对象是否是函数，则需要使用`types`模块：

```powershell
>>> import types
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType
True
>>> type(abs)==types.BuiltinFunctionType
True
>>> type(lambda x: x)==types.LambdaType
True
>>> type((x for x in range(10)))==types.GeneratorType
True
```

#### instance

---

`type`获取类型较为方便，但是对于继承关系来说，就没有那么方便了，此时需要用到instance，例如对于如下继承链：

```
object -> Animal -> Dog -> Husky
```

```powershell
>>> a = Animal()
>>> d = Dog()
>>> h = Husky()
>>> isinstance(h, Husky)
True
>>> isinstance(h, Dog)
True
>>> isinstance(h, Animal)
True
>>> isinstance(d, Dog) and isinstance(d, Animal)
True
>>> isinstance(d, Husky)
False
>>> isinstance('a', str)
True
>>> isinstance(123, int)
True
>>> isinstance(b'a', bytes)
True
>>> isinstance([1, 2, 3], (list, tuple))
True
>>> isinstance((1, 2, 3), (list, tuple))
True
```

{% note info, 总是优先使用isinstance()判断类型，可以将指定类型及其子类“一网打尽”。 %}

### 获取对象属性

---

如果想要获取一个对象的所有属性和方法，可以使用`dir()`函数，它将返回一个包含该对象的所有属性名的字符串`list`

```powershell
>>> dir('ABC')
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'removeprefix', 'removesuffix', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```

其中类似`__len__`的方法再python中是有特殊用途的，如果调用`len()`函数试图获取一个对象的长度，实际上，在`len()`函数试图获取一个对象的长度，实际上，在`len()`函数内部是去调用该对象的`__len__()`方法，所以以下两行代码是等价的：

```python
>>> len('ABC')
3
>>> 'ABC'.__len__()
3
```

这就意味着自定义类如果也想通过`len`函数获取长度，则只需要在我们自定义的类中实现`__len__()`方法即可：

```python
class MyDog(object):
    def __len__(self):
        retrun 100

dog = MyDog()
len(dog)  # 100
```

此外配合`getattr()`，`setattr()`，`hasattr()`，可以直接操作一个对象的状态：

```python
class MyObject(object):
     def __init__(self):
         self.x = 9
     def power(self):
         return self.x * self.x
        

obj = MyObject()
hasattr(obj,'x')  # True 有属性x
hasattr(obj,'y')  # False 没有属性y
setattr(obj,'y',19)  # 设置一个属性y
hasattr(obj,'y')  # True 有属性y
getattr(obj,'y')  # 19 获取属性y
```

如果获取一个没有的属性，则会报如下错误：

```powershell
>>> getattr(obj, 'z') # 获取属性'z'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'MyObject' object has no attribute 'z'
```

`getattr()`还支持自定义错误返回值：

```powershell
>>> getattr(obj, 'z', 404) # 获取属性'z'，如果不存在，返回默认值404
404
```

此外，对象的方法也是可以操作的：

```powershell
>>> hasattr(obj, 'power') # 有属性'power'吗？
True
>>> getattr(obj, 'power') # 获取属性'power'
<bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
>>> fn = getattr(obj, 'power') # 获取属性'power'并赋值到变量fn
>>> fn # fn指向obj.power
<bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
>>> fn() # 调用fn()与调用obj.power()是一样的
```

这些操作在不确定自己获得的是一个怎样的对象时会派上用场：

```python
def readImage(fp):
    if hasattr(fp, 'read'):
        return readData(fp)
    return None
```

### 实例属性和类属性

---

python中类的属性有实例属性与类属性的区别

实例属性是，在创建实例时，为每个实例都增加上的属性，操作如下：

```python
class Student(object):
    def __init__(self, name):
        self.name = name

s = Student('Bob')
s.score = 90
```

而类属性则是为类绑定的属性：

```python
class Student(object):
    name = 'Student'
```

这样的属性我们不实例化也可以访问到：

```powershell
>>> Student.name
Student
>>> s = Student()
>>> s.name
Student
>>> s.name = "Michael"
>>> s.name
Michael
>>> Student.name
Student
>>> del s.name
>>> s.name
Student
```

可以看到，在类和对象具有同名属性时，我们访问对象的该属性，优先访问到的是实例属性，因此不要对实例属性和类属性使用相同的名字，因为相同名称的实例属性将屏蔽掉类属性。

### 属性绑定

---

由于Python动态语言的特性，Python可以轻松实现在允许过程中对类进行操作，比如可以很轻松的为类绑定方法与属性：

```python
class Student(object):
    pass
```

```powershell
>>> s = Student()
>>> s.name = 'Michael' # 动态给实例绑定一个属性
>>> print(s.name)
Michael
```

绑定方法需要用到`types`库：

```powershell
>>> def set_age(self, age): # 定义一个函数作为实例方法
...     self.age = age
...
>>> from types import MethodType
>>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
>>> s.set_age(25) # 调用实例方法
>>> s.age # 测试结果
25
```

与属性相同，为特定对象绑定的方法无法在另一对象或类中使用，需要为类绑定方法才能让所有对象均可使用

#### `__slots__`

python还提供了一种可以限制运行时绑定的属性的操作，比如只允许动态的为`Student`实例添加`name`和`age`属性，此时就可以用到`__slots__`变量：

```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

然后尝试为其绑定属性：

```powershell
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```

可见由于`score`没有被`__slots__`指明，因此绑定此属性时报错。

但`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的：

```powershell
>>> class GraduateStudent(Student):
...     pass
...
>>> g = GraduateStudent()
>>> g.score = 9999
```

除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。

### @property

---

在进行面向对象编程时，我们为了代码的健壮性，通常会对某个类的属性操作进行封装，让这些属性变成私有属性，然后通过对外暴露`getter`和`setter`方法来操作这些属性，这样我们在修改和访问这些属性时，就能增加一些类似类型检测，安全检测等诸如此类的操作。

通常我们可以这样写：

```python
class Student(object):

    def get_score(self):
         return self._score

    def set_score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

修改和查询`score`属性时，我们需要这样做：

```powershell
>>> s = Student()
>>> s.set_score(60) # ok!
>>> s.get_score()
60
>>> s.set_score(9999)
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!
```

访问`score`属性时，我们需要通过`get_score`函数进行访问。

python为我们提供了更为直观的访问方法，可以直接通过`s.score`进行访问和修改，并且还能实现如上的安全检测功能，这就是`@property`语法糖：

```python
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

把一个getter方法变成属性，只需要加上`@property`就可以了，此时，`@property`本身又创建了另一个装饰器`@score.setter`，负责把一个setter方法变成属性赋值，于是，我们就拥有一个可控的属性操作：

```powershell
>>> s = Student()
>>> s.score = 60 # OK，实际转化为s.set_score(60)
>>> s.score # OK，实际转化为s.get_score()
60
>>> s.score = 9999
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!
```

还可以实现对属性的只读访问：

```python
class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2021 - self._birth
```

上面的`age`属性并没有设置`setter`方法，因此感官上来说是不能对其进行修改操作。

需要特别注意的是：

{% note danger, 属性的方法名和实例变量名不能一样 %}

看如下例子：

```python
class Student(object):

    # 方法名称和实例变量均为birth:
    @property
    def birth(self):
        return self.birth
```

这时如果我们调用`s.birth`，首先执行上方定义的方法，在执行到`return self.birth`，又视为调用了对象`s`的`score`属性，于是又转到`birth`方法，形成无限层递归，由于没有对尾递归进行优化，最终将报栈溢出错误`RecursionError`

### MixIn

---

Python中的类是支持多继承的，而MixIn设计思路是Python中为了更好的实现Python多继承的设计思路。

现在我们需要为如下几种动物创建类：

1. Dog - 狗勾🐕
2. Bat - 蝙蝠🦇
3. Parrot - 鹦鹉🦜
4. Ostrich - 鸵鸟🦩

如果我们想要将

```ascii
                ┌───────────────┐
                │    Animal     │
                └───────────────┘
                        │
           ┌────────────┴────────────┐
           │                         │
           ▼                         ▼
    ┌─────────────┐           ┌─────────────┐
    │   Mammal    │           │    Bird     │
    └─────────────┘           └─────────────┘
           │                         │
     ┌─────┴──────┐            ┌─────┴──────┐
     │            │            │            │
     ▼            ▼            ▼            ▼
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│  MRun   │  │  MFly   │  │  BRun   │  │  BFly   │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
     │            │            │            │
     │            │            │            │
     ▼            ▼            ▼            ▼
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│   Dog   │  │   Bat   │  │ Ostrich │  │ Parrot  │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
```

在Java中采用单继承的方式，如果必须为每个种类定义一个类的话，想必定义这些类就需要一长串的代码。但是通常Java中会使用接口来解决这一问题。Python中则使用多继承：

```python
class Animal(object):
    pass

# 大类:
class Mammal(Animal):
    pass

class Bird(Animal):
    pass


class RunnableMixIn(object):
    def run(self):
        print('Running...')

class FlyableMixIn(object):
    def fly(self):
        print('Flying...')
        
# 各种动物:
class Dog(Mammal, RunnableMixIn):
    pass

class Bat(Mammal, FlyableMixIn):
    pass

class Parrot(Bird, FlyableMixIn):
    pass

class Ostrich(Bird, RunnablebleMixIn):
    pass
```

其中以`MixIn`结尾的类的目的就是给一个类增加多个功能，这一在设计类的时候，我们优先考虑通过多重继承来组合多个`MixIn`的功能，而不是设计多层次复杂的继承关系。

可见`MixIn`在思想上有些类似java中的接口，但是具体操作和实现上有很大的区别。

那么为什么java没有多继承机制呢？

因为采用多继承时，如果继承的两个类中有同名方法，那么调用该方法时编译器将不知道调用的是哪个父类中的方法。

那么Python中又是通过怎样的方式解决的呢？

实验过程参考如下博客：

{% link, Python多重继承排序原理, https://www.jianshu.com/p/c9a0b055947b %}

该博客通过介绍拓扑排序以及C3算法，最后经过举例验证，得出结论：python中多继承的方法访问顺序遵从拓扑序。

该博客还补充了一个概念：`MRO`即*method resolution order*

对于如下继承关系：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
class A(object):
    def foo(self):
        print('A foo')
    def bar(self):
        print('A bar')

class B(object):
    def foo(self):
        print('B foo')
    def bar(self):
        print('B bar')

class C1(A,B):
    pass

class C2(A,B):
    def bar(self):
        print('C2-bar')

class D(C1,C2):
    pass

if __name__ == '__main__':
    print(D.__mro__)
    d=D()
    d.foo()
    d.bar()
```

上述继承关系的图如下：

![img](https://upload-images.jianshu.io/upload_images/143920-c35a0535a4e124a3.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/349/format/webp)

当我们调用`d.foo()`时，按照拓扑排序规则，解释器会先去寻找D类中是否拥有`foo()`方法，即在图中去掉D对应的点以及D的出边。

于是存在的点只剩下：[C1,C2,A,B,object]，其中没有入度的点为C1和C2，根据左优先原则，寻找C1中是否包含`foo()`方法，去掉C1点以及C1的出边。

此时存在的点只剩下：[C2,A,B,object]，其中没有入度的点为C2，寻找C2中是否包含`foo()`方法，去掉C2点以及C2的出边。

此时存在的点只剩下：[A,B,object]，其中没有入度的点为A，B，根据左优先原则，寻找A中是否包含`foo()`方法，发现包含，于是调用该方法输出`A foo`

调用`d.bar()`时，同样也是按照拓扑序进行查找，最终输出`C2-bar`

而第26行代码，调用了D类的`__mro__`方法，将直接输出解释器寻找方法的先后顺序。因此最终的测试结果为：

```python
(<class '__main__.D'>, <class '__main__.C1'>, <class '__main__.C2'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
A foo
C2-bar
```

### 定制类

---

与`__slots__`类似，Python还具有很多类似命名的，有特殊作用的函数以及变量。下面来积累几个

#### `__str__`

---

类似Java中的`toString()`方法，当对某个实例进行打印时，实际上就是调用该实例的`__str__`方法：

```python
class Student(object):
	def __init__(self, name):
        self.name = name
    def __str__(self):
        return 'Student object (name: %s)' % self.name


print(Student('Michael'))
# Student object (name: Michael)
```

#### `__repr__`

---

用于显示对该对象的解释，即我们直接在命令行中输入对象名时显示的内容，就是通过调用对象的`__repr__`方法得到，通常情况下，我们会将`__repr__`方法与`__str__`方法设置为同一个：

未定义`__repr__`时：

```powershell
>>> s = Student('Michael')
>>> s
<__main__.Student object at 0x109afb310>
```



```python
class Student(object):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return 'Student object (name=%s)' % self.name
    __repr__ = __str__
```

定义后：

```powershell
>>> s = Student('Michael')
>>> s
Student object (name: Michael)
```

#### `__iter__`&`__next__`

---

如果我们需要使用`for...in`来遍历某个对象，我们就需要实现`__iter__`与`__next__`方法。

1. `__iter__`用于返回一个迭代对象。即`for n in object`中的`n`
2. `__next__`用于在循环中反复调用1中返回的迭代对象的该函数以得到下一个对象，知道遇到`StopIteration`错误时退出循环。

例如我们写一个可遍历的斐波那契类：

```python
class Fib(object):
    def __init__():
        self.a, self.b = 0, 1
    def __iter__(self):
        return self
    def __next__(self):
        self.a, self.b = self.b, self.a + self.b
        if self.a > 100000:
            raise StopIteration()
        return self.a
```

```powershell
>>> for n in Fib():
...    print(n)
...
1
1
2
3
5
...
46368
75025
```

#### `__getitem__`

---

用于对对象进行索引访问，如：

```powershell
>>> Fib()[5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'Fib' object does not support indexing
```

该操作实际上时调用对象的`__getitem__`方法，并将索引当作参数传入。当我们实现该方法后：

```python
class Fib(object):
    def __init__():
        self.a, self.b = 0, 1
    def __iter__(self):
        return self
    def __next__(self):
        self.a, self.b = self.b, self.a + self.b
        if self.a > 100000:
            raise StopIteration()
        return self.a
    def __getitem__(self, n):
        a, b = 1, 1
        for x in range(n):
            a,b = b, a + b
        return a
```

```powershell
>>> f = Fib()
>>> f[0]
1
>>> f[1]
1
>>> f[5]
8
>>> f[10]
89
>>> f[100]
573147844013817084101
```

但是单纯的这样写，无法支持切片操作，切片操作实际上也是调用了对象的`__getitem__`方法，但是传入的是`slice`切片对象。由于动态语言python并不具备重载的能力，我们需要函数内部通过`if`进行判断

```python
class Fib(object):
    def __init__():
        self.a, self.b = 0, 1
    def __iter__(self):
        return self
    def __next__(self):
        self.a, self.b = self.b, self.a + self.b
        if self.a > 100000:
            raise StopIteration()
        return self.a
    def __getitem__(self, n):
        if isinstance(n, int):  # 接受int型参数
	        a, b = 1, 1
    	    for x in range(n):
        	    a,b = b, a + b
	        return a
        if isinstance(n, slice):  # 接受slice型参数
            start = n.start
            stop = n.stop
            if start is None:
                start = 0
            a, b = 1, 1
            L = []
            for x in range(stop):
                if x >= start:
                    L.append(a)
                a, b = b, a + b
            return L
```

```powershell
>>> f = Fib()
>>> f[0:5]
[1, 1, 2, 3, 5]
>>> f[:10]
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
```

如上并没有对`step`参数和负数进行处理，实际上这些都是可以处理的。

另外，如果将对象看作是`dict`，那么在该方法内还需要实现对`object`类型的参数第处理。

此外，除了`__getitem__`，还有`___setitem__`和`__delitem__`方法，用于删除和修改某个元素。

因此我们可以让自己的类创建的对象与`list`、`tuple`、`dict`没有什么区别。这都要归功于动态语言的`鸭子类型`

#### `__getattr__`

---

该方法当我们访问某个对象中的未定义的属性时调用：

```python
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr=='score':
            return 99
```

当调用不存在的属性时，比如`score`，Python解释器会试图调用`__getattr__(self, 'score')`来尝试获得属性，这样，我们就有机会返回`score`的值：

```powershell
>>> s = Student()
>>> s.name
'Michael'
>>> s.score
99
>>> s.age
None
```

当访问没有定义的属性`score`和`age`时，就会调用`__getattr__`函数，由于该函数中未包含对`age`的处理，于是返回了默认返回值None。

要让class只响应特定的几个属性，我们就要按照约定，抛出`AttributeError`的错误：

```python
class Student(object):

    def __getattr__(self, attr):
        if attr=='age':
            return lambda: 25
        raise AttributeError('\'Student\' object has no attribute \'%s\'' % attr)
```

返回函数也是完全可以的：

```python
class Student(object):

    def __getattr__(self, attr):
        if attr=='age':
            return lambda: 25
```

只是调用方式要变为：

```python
>>> s.age()
25
```

该方法在写SDK时运用广泛。

有时我们可能需要给每个URL对应的API都写一个方法，API一旦改动，SDK也要改，利用动态的`__getattr__`，我们可以写一个链式调用：

```python
class Chain(object):
    def __init__(self, path=''):
        self._path = path
        
    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))
    
    def __str__(self):
        return self._path
    
    __repr__ = __str__
```

这样我你们进行如下调用时，就能自由的获取各式各样的链接：

```powershell
>>> Chain().status.user.timeline.list
'/status/user/timeline/list'
```

每次访问未定义参数时，都会调用对象的`__getattr__`方法，在此方法内，我们使用传入对象的`_path`以及传入的参数创建一个`Chain`对象，并返回，这样就能与后面属性调用组合，形成链式调用。

#### `__call__`

---

当我们需要直接对实例进行函数调用时，就会调用`__call__`函数：

```python
class Student(object):
    def __init__(self, name):
        self.name = name

    def __call__(self):
        print('My name is %s.' % self.name)
```

```powershell
>>> s = Student('Michael')
>>> s() # self参数不要传入
My name is Michael.
```

有了这个参数，那么完全可以把对象看成函数，把函数看成对象。

此处补充一个区分对象与函数的方法：`Callable`

```powershell
>>> callable(Student())
True
>>> callable(max)
True
>>> callable([1, 2, 3])
False
>>> callable(None)
False
>>> callable('str')
False
```

可见，函数即“可调用”对象。

有时我们会看到一些将参数放入URL中的REST API，比如GitHub的API：

```http
Get /users/:user/repos
```

调用时，我们需要把`:user`替代为实际用户名，我们需要写出如下调用式：

```python
Chain().users('michael').repos
```

此时在之前的链式调用的基础上，我们看到中间多出来一个类似函数调用的形式，因此我们需要将其中一环变成可调用对象，就需要用到我们的`__call__`，我们可以这样实现：

```python
class Chain(object):
    def __init__(self, path=''):
        self._path = path
        
    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))
    
    def __str__(self):
        return self._path
    
    def __call__(self, users):
        return Chain('%s/%s' % (self.path, users))
    
    __repr__ = __str__
```

这样最外层的`Chain()`创建了一个对象，访问`users`属性，由于不存在该属性，进入`__getattr__`方法，并返回一个新的对象，该对象与之后的`('michael')`构成函数调用，于是调用该对象的`__call__`方法，再次返回一个新对象，最后与`repos`构成访问属性。最终完成链式调用。

### 枚举类型

---

与java一样，python也提供枚举类型。

引入`enum`类后即可创建：

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
```

这样我们就可以这样访问：

```python
for name,member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)

# Jan => Month.Jan , 1
# Feb => Month.Feb , 2
# Mar => Month.Mar , 3
# Apr => Month.Apr , 4
# May => Month.May , 5
# Jun => Month.Jun , 6
# Jul => Month.Jul , 7
# Aug => Month.Aug , 8
# Sep => Month.Sep , 9
# Oct => Month.Oct , 10
# Nov => Month.Nov , 11
# Dec => Month.Dec , 12
```

其中`__members__`是Month中的一个特殊属性，该属性的类型是`mappingproxy`，具有如下特性：

```python

# 不可变映射类型,(字典)MappingProxyType
   
# python3.3开始,types模块中引入了一个封装类名叫MappingProxyType
# 如果给这个类一个映射,它会返回一个只对映射视图.
# 虽然是个只读的视图,但是它是动态的,这意味着如果对原映射做出了改动,
# 我们可以通过这个视图观察到,但是无法通过这个视图对原映射做出修改
   
 

#示例
from types import MappingProxyType
#创建一个集合
index_a = {'a' : 'b'}
#创建index_a的映射视图
a_proxy = MappingProxyType(index_a)
print(a_proxy)
a_proxy['a']
# #不能对a_proxy视图进行修改
# a_proxy['b'] = 'bb'
#但是可以对原映射进行修改
index_a['b'] = 'bb'
print(a_proxy)
 
# {'a': 'b'}
# {'a': 'b', 'b': 'bb'}

```



如果需要更精确的控制枚举类型，可以用一个派生自`Enum`的自定义类创建：

```python
for enum import Enum

@unique
class Weekday(Enum):
    sun = 0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```

其中装饰器`@unique`用以检查并保证没有重复值

这样创建的枚举类型可以用如下方式访问：

```powershell
>>> day1 = Weekday.Mon
>>> print(day1)
Weekday.Mon
>>> print(Weekday.Tue)
Weekday.Tue
>>> print(Weekday['Tue'])
Weekday.Tue
>>> print(Weekday.Tue.value)
2
>>> print(day1 == Weekday.Mon)
True
>>> print(day1 == Weekday.Tue)
False
>>> print(Weekday(1))
Weekday.Mon
>>> print(day1 == Weekday(1))
True
>>> Weekday(7)
Traceback (most recent call last):
...
ValueError: 7 is not a valid Weekday
>>> for name, member in Weekday.__members__.items():
...     print(name, '=>', member)
...
Sun => Weekday.Sun
Mon => Weekday.Mon
Tue => Weekday.Tue
Wed => Weekday.Wed
Thu => Weekday.Thu
Fri => Weekday.Fri
Sat => Weekday.Sat
```

###  元类

---

动态语言与静态语言的差别在于函数于类的定义，动态语言并不是在编译时创建类，而是在运行时创建类。比如如下类的创建：

```python
class Hello(object):
    def hello(self, name='world'):
        print('Hello. %s.' % name)
```

我们可以将该类写到一个模块`hello.py`，再通过另一个模块来引入该模块，查看类创建的效果：

```python
from hello import Hello

h = Hellp()
h.hello()
print(type(Hello))
print(type(h))
```

引入模块时，python解释器就会依次执行模块中的所有语句，执行`hello.py`的结果就是创建了一个`Hello`对象。为什么说是对象呢。`Hello`明明是类啊。Python中，万物接对象，包括我们创建的类。我们创建的类，实际上也是`type`类的一个对象。

从上例的输出结果可以看出：

```python
Hello, world.
<class 'type'>
<class 'hello.Hello'>
```

`Hello`是一个class，它的类型就是`type`，而`h`是一个实例，它的类型就是class `Hello`。

`type()`函数出了可以看到对象的类型外，还可以用于动态的创建类：

```python
def fn(self, name = 'world'):
    print('Hello, %s.' % name)

Hello = type('Hello', (object,), dict(hello = fn))
h = Hello()
h.hello()
print(type(Hello))
print(type(h))
```

输出结果：

```python
Hello, world.
<class 'type'>
<class 'hello.Hello'>
```

利用`type()`动态的创建类时，需要提供三个参数：

1. class的名称。
2. 继承的父类集合，需要传入tuple，注意tuple的单元素写法。
3. class的方法名称于函数进行绑定，此处将方法`fn`绑定到`hello`上

通过`type()`函数创建的类和直接写class是完全一样的，因为Python解释器遇到class定义时，仅仅是扫描一下class定义的语法，然后调用`type()`函数创建出class。

可见，在动态操作类这件事上，动态语言比如python比静态语言，比如java方便很多。

除了使用type外，还能使用`metaclass`对类的创建进行控制

#### metaclass

---

`metaclass`直译为原类。

实例的创建可由类控制，那么类的创建则是由元类进行控制。

下面引用一个例子：

定义一个`metaclass`可以给我们自定义的`MyList`增加一个`add`方法：

定义`ListMetaclass`，按照默认习惯，`metaclass`的类名总是以`Metaclass`结尾，以便清楚地表示这是一个`metaclass`：

```python
# metaclass是类的模板，所以必须从`type`类型派生：
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)
```

接下来我们利用这个原类，来创建我们自己的`list`类：

```python
class MyList(list, metaclass = ListMetaclass):
    pass
```

此时，在定义类时，我们指定使用`ListMetaclass`来定制类，传入关键字参数`metaclass`

此后，Python解释器在创建`MyList`时，要通过`ListMetaclass.__new__()`来创建，在此，我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。

`__new__()`方法接收到的参数依次是：

1. 当前准备创建的类的对象
2. 类的名字
3. 类继承的父类集合
4. 类方法集合

下面我们试着创建一个`MyList`对象：

```powershell
>>> L = MyList()
>>> L.add(1)
>> L
[1]
```

普通的`list`并没有`add()`方法。

动态的修改类的定义将在编写ORM中起到非常大的作用。

`ORM`——‘Object Relational Mapping’，即对象——关系映射。也即是将数据库中的一个表于一个类对应，一行与一个对象对应。

我们尝试利用`metaclass`来实现ORM中的保存功能：

1. 编写底层模块的第一步，就是先把调用接口写出来。比如，使用者如果使用这个ORM框架，想定义一个`User`类来操作对应的数据库表`User`，我们期待他写出这样的代码：

```python
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

# 创建一个实例：
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库：
u.save()
```

其中，父类`Model`和属性类型`StringField`、`IntegerField`是由ORM框架提供的，剩下的方法比如`save()`全部由父类`Model`自动完成。虽然`metaclass`的编写会比较复杂，但ORM的使用者用起来却异常简单。

2. 接下来我们实现`Field`类，负责保存数据库表的字段名和字段类型：

```python
class Field(object):

    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type

    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)
```

在此基础上，定义各种类型的子类：

```python
class StringField(Field):

    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')  # 调用父类的__init__方法

class IntegerField(Field):

    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')
```

3. 接下来编写`ModelMetaclass`：

```python
class ModelMetaclass(type):

    def __new__(cls, name, bases, attrs):
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model: %s' % name)
        mappings = dict()
        for k, v in attrs.items():
            if isinstance(v, Field):
                print('Found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
        for k in mappings.keys():
            attrs.pop(k)
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = name # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)
```

4. 使用元类`ModelMetaclass`创建基类`Model`：

```python
class Model(dict, metaclass=ModelMetaclass):

    def __init__(self, **kw):
        super(Model, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.items():
            fields.append(v.name)
            params.append('?')
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))
```

当用户定义一个`class User(Model)`时，Python解释器首先在当前类`User`的定义中查找`metaclass`，如果没有找到，就继续在父类`Model`中查找`metaclass`，找到了，就使用`Model`中定义的`metaclass`的`ModelMetaclass`来创建`User`类，也就是说，`metaclass`可以隐式地继承到子类，但子类自己却感觉不到。

在`ModelMetaclass`中，一共做了几件事情：

1. 排除掉对`Model`类的修改；
2. 在当前类（比如`User`）中查找定义的类的所有属性，如果找到一个Field属性，就把它保存到一个`__mappings__`的`dict`中，同时从类属性中删除该`Field`属性，否则，容易造成运行时错误（实例的属性会遮盖类的同名属性）；
3. 把表名保存到`__table__`中，这里简化为表名默认为类名。

在`Model`类中，就可以定义各种操作数据库的方法，比如`save()`，`delete()`，`find()`，`update`等等。

使用如下代码进行测试：

```python
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
u.save()
```

输出如下：

```
Found model: User
Found mapping: email ==> <StringField:email>
Found mapping: password ==> <StringField:password>
Found mapping: id ==> <IntegerField:uid>
Found mapping: name ==> <StringField:username>
SQL: insert into User (password,email,username,id) values (?,?,?,?)
ARGS: ['my-pwd', 'test@orm.org', 'Michael', 12345]
```

这样一来我们就能直接得到对应操作的`SQL`语句，允许即可。

**Table of Contents**


   * [Python Interview](#python_interview)
      * [1 Python语言特性](#1-python语言特性)
      * [2 鸭子类型](#2-鸭子类型)
      * [3 猴子补丁](#3-猴子补丁)
      * [4 Python中对象自省](#4-Python对象自省)
      * [5 Python中函数参数传递](#5-Python中函数参数传递)
      * [6 *args和**kwargs](6-*args和**kwargs)
      * [7 装饰器](7-装饰器)
      * [8 ==和is的区别](8-==和is的区别)
      * [9 默认做浅拷贝](9-默认做浅拷贝)
      * [10 classmethod和staticmethod](10-classmethod和staticmethod)
      * [11 Python的私有属性和受保护的属性](11-Python的私有属性和受保护的属性)
      * [12 新式类和旧式类](12-新式类和旧式类)
      * [13 new和init](13-new和init)
      * [14 元类](14-元类)
      * [15 单例模式](15-单例模式)
      * [16 可迭代对象&迭代器&生成器](16-可迭代对象&迭代器&生成器)

<!-- markdown-toc end -->



# python_interview

## 1 Python语言特性

python是**动态**、**强类型**、**解释型语言**：

-   解释型 & 编译型

    >解释型语言：在代码运行时，解释器将代码逐行翻译成机器语言，代码可以按行运行(实时翻译)；
    >
    >编译型语言：先通过编译器对代码进行编译，生成可执行文件，然后运行可执行文件(翻译书籍的过程)；
    >
    >**优缺点：**
    >
    >-   解释型语言相当于实时翻译，省去了编译这个步骤。每运行一次代码，解释器都要重新翻译代码，而解释器把代码翻译成机器语言是要花时间的，所以解释型语言的运行效率比编译型语言低很多，速度慢，**性能上处于劣势**；
    >-   编译型语言相当于提前翻译，运行时直接运行可执行文件。编译只需要一次，后面不管运行多少次，都可以直接运行可执行文件，所以，编译型语言的运行效率比解释型语言高很多，速度快，**性能上有优势**。 
    >
    >因为性能上的劣势，解释型语言不适合开发操作系统、大型应用程序、数据库系统等对性能要求高的程序，这些程序基本都是使用编译型语言。当然，解释型语言可以通过对解释器进行优化来提高运行效率，同时，随着计算机硬件的性能提升，也可以弥补解释型语言在性能上的不足，使性能差距对生产的影响降低，甚至消除。

-   动态 & 静态类型语言

    >动态类型和静态类型语言的区别在于是在运行时还是编译期**确定数据类型**；
    >
    >**优缺点：**
    >
    >-   动态类型的语言不需要提前声明变量的类型，使得**代码更加简洁**，但是也带来了很多弊端，例如一些变量定义得不规范，会使代码**可读性降低**，**增加维护代码的难度**。

-   强类型 & 弱类型语言

    >强类型(例如Python)和弱类型(例如JavaScript)语言的区别在于是否会发生隐式类型转换；
    >
    >在动态类型和静态类型方面，Python给程序员留了足够的自由，但在强类型和弱类型方面，Python又对数据的运算进行了限制。可以猜想，Guido这么设计是经过深思熟虑的，在定义变量的时候给了足够自由，也提升了代码的简洁性，但在进行**数据运算时进行了限制**，加了一道保险，避免太过自由带来一些不必要的问题。



## 2 鸭子类型

>不要检查它是不是鸭子；检查它的叫声像不像鸭子、它走路的姿势像不像鸭子，等等。具体检查什么取决于你想使用语言的哪些行为。
>
>​																																															  ---- Alex Marteli

在动态类型语言中，有“鸭子类型”这个术语，它关注的是**对象的行为**，而不是对象的类型。它让动态语言更加灵活，而不需要去实现一大堆模式。

示例1：在Python中天然支持多态

```python
class Cat:
    def say(self):
        print("I am a cat")


class Dog:
    def say(self):
        print("I am a dog")


class Duck:
    def say(self):
        print("I am a duck")


animals_list = [Cat, Dog, Duck]
for animal in animals_list:
    animal().say()  # 关注animal对象是否实现了say方法,而不在意其类型
```

示例2：list.extend()方法，对接受的参数关注的是否是可迭代的对象：

```python
    def extend(self, *args, **kwargs): # real signature unknown
        """ Extend list by appending elements from the iterable. """
        pass
```

示例：序列类型对象只需要实现 `__len__`和`__getitem__` 两个魔法方法，我们就可以说它是序列，因为它的行为像序列；

示例4：例如file、StringIO、socket对象在Python文档中都叫“文件类对象”，因为它们行为基本与文件一致，实现了部分文件接口（read、write），满足上下文（with语句）相关需求等。

## 3 猴子补丁

猴子补丁指的是在**运行时修改类或者模块，而不改动源码**。猴子补丁很强大，但是打补丁的代要与打补丁的程序耦合十分紧密；在Python中打猴子补丁不难，但是也有局限，Python不允许为内置类型打猴子补丁，因为这样可以确保str对象的方法始终是那些，能够减少外部库打的补丁有冲突的概率。

例如常用的并发库gevent需要修改内置的socket模块（阻塞->非阻塞）：

```python
>>> import socket
>>> socket.socket
<class 'socket.socket'>
# "after monkey patch"
>>> from gevent import monkey
>>> monkey.patch_socket()
>>> socket.socket
<class 'gevent._socket3.socket'>

>>> import select
>>> select.select
<built-in function select>
# "after monkey patch"
>>> monkey.patch_select
>>> select.select
<function select at 0x7fecb02fe6a8>
```



## 4 Python对象自省

自省（Introspection）是动态语言又一强大的特性，可以在运行时通过一定的机制查询到对象的内部结构；例如`dir()`、`isinstance()` 等。



## 5 Python中函数参数传递

from: [taizilongxu/interview_python](https://github.com/taizilongxu/interview_python)，对其进行了主要对考察的知识点&知识点延伸进行了补充。

看两个例子:

```python
a = 1
def fun(a):
    a = 2
fun(a)
print a  # 1
```

```python
a = []
def fun(a):
    a.append(1)
fun(a)
print a  # [1]
```

所有的变量都可以理解是内存中一个对象的“引用”，或者，也可以看似c中void*的感觉。

通过`id`来看引用`a`的内存地址可以比较理解：

```python
a = 1
def fun(a):
    print "func_in",id(a)   # func_in 41322472
    a = 2
    print "re-point",id(a), id(2)   # re-point 41322448 41322448
print "func_out",id(a), id(1)  # func_out 41322472 41322472
fun(a)
print a  # 1
```

注：具体的值在不同电脑上运行时可能不同。

可以看到，在执行完`a = 2`之后，`a`引用中保存的值，即内存地址发生变化，由原来`1`对象的所在的地址变成了`2`这个实体对象的内存地址。

而第2个例子`a`引用保存的内存值就不会发生变化：

```python
a = []
def fun(a):
    print "func_in",id(a)  # func_in 53629256
    a.append(1)
print "func_out",id(a)     # func_out 53629256
fun(a)
print a  # [1]
```

这里记住的是类型是属于对象的，而不是变量。而对象有两种,“可更改”（mutable）与“不可更改”（immutable）对象。在python中，strings, tuples, 和numbers是不可更改的对象，而 list, dict, set 等则是可以修改的对象。(这就是这个问题的重点)

当一个引用传递给函数的时候,函数自动复制一份引用,这个函数里的引用和外边的引用没有半毛关系了.所以第一个例子里函数把引用指向了一个不可变对象,当函数返回的时候,外面的引用没半毛感觉.而第二个例子就不一样了,函数内的引用指向的是可变对象,对它的操作就和定位了指针地址一样,在内存里进行修改.

如果还不明白的话,这里有更好的解释: http://stackoverflow.com/questions/986006/how-do-i-pass-a-variable-by-reference

***

**知识点考察：**

-   Python支持的参数传递模式：Python中唯一支持的参数传递模式是**共享传参(call by sharing)**，共享传参指的是函数的各个形式参数获得实参中各个**引用的副本**；也就是说，函数中**形参是实参的别名**;

-   Python的变量：变量是标注（引用式变量），而不是盒子；最好把变量理解为附加在对象上的便利贴更贴切；

    ```python
    a = [1, 2, 3]
    b = a # b是a的别名,绑定同一个对象
    a.append(4)
    print(b) # [1, 2 ,3 ,4]
    ```

-   可变类型对象和不可变类型对象

    >可变类型对象：list、dict、set
    >
    >不可变类型对象：int、str、bool、tuple

-   Python中的赋值语句执行顺序是什么？

    先来看一个示例：

    ```python
    class Gizmo:
        def __init__(self) -> None:
            print('Gizmo id: {}'.format(id(self)))  # 实例化对象时,打印对象的标识(内存地址)
    
    
    x = Gizmo() # Gizmo id: 140650247909160
    # 先打印：Gizmo id: 140650248064696
    # 然后抛出异常:TypeError: unsupported operand type(s) for *: 'Gizmo' and 'int'
    y = Gizmo() * 10
    ```

    从上面的例子中可以看出Python中赋值语句，应该始终先读右边。**对象在右边先创建或获取，在此之后左边的变量才会绑定到对象上**。



**知识点延伸考察：**

-   **可变类型作为函数参数的默认值**

```python
class HauntedBus:
    """ 定义一个Bus类 """
    def __init__(self, passengers=[]):
        self.passengers = passengers
    
    def pick(self, name):
        self.passengers.append(name)
        
    def drop(self, name):
		self.passengers.remove(name)
        
        
        
>>> bus1 = HauntedBus(['alice', 'lily'])
>>> bus1.passengers
['alice', 'lily']
>>> bus1.pick('zgt')
>>> bus1.drop('alice')
>>> bus1.passengers  # bus1工作正常,没有出现异常
['lily', 'zgt']
>>> bus2 = HauntedBus() # bus2为空,把默认的空列表赋值给self.passengers
>>> bus2.passengers
[]
>>> bus2.pick('jimi')
>>> bus2.passengers
['jimi']
>>> bus3 = HauntedBus() # bus3和bus2一样为空
>>> bus3.passengers 
['jimi'] # 这里出现了异常,在bus2中的jimi出现在了bus3
>>> bus3.pick('bill')
>>> bus3.passengers
['jimi', 'bill']
>>> bus2.passengers
['jimi', 'bill'] # 通过例子可以看出,bus2.passenger和bus3.passengers会共享同一个乘客列表（默认参数的空列表）
```

如果函数中定义了默认值参数，默认值会在定义函数时计算（通常在加载模块时），默认值会变成函数对象的属性。因此如果默认值是可变类型且修改了它的值的话，那么后续函数的调用可能会受影响。

```python
# 可以通过dir查看HauntedBus.__init__对象的属性
>>> dir(HauntedBus.__init__)
['__annotations__', '__call__', '__defaults__', ...]
# 查看__defaults__属性的值
>>> HauntedBus.__init__.__defaults__
(['jimi', 'bill'],)
# 最后可以证明下bus2.passengers是一个别名，绑定到HauntedBus.__init__.__defaults__属性的第一个元素对象上
>>> HauntedBus.__init__.__defaults__[0] is bus2.passengers
True
>>> HauntedBus.__init__.__defaults__[0] is bus3.passengers
True
```

-   如果业务中函数的默认参数需要接受可变类型参数，该如何防御？

```python
class HauntedBus:
    """ 定义一个Bus类 """
    def __init__(self, passengers=None):
        if passengers is None:
	        self.passengers = [] # 当没有传入passengers实参时,创建一个空列表
        else:
            self.passengers = list(passengers) # 当有传入passengers实参时,生成passengers的副本,如果不是列表则把它转换成列表
    
    def pick(self, name):
        self.passengers.append(name)
        
    def drop(self, name):
		self.passengers.remove(name)
```



## 6 *args和**kwargs

当函数中遇到可变参数时，可以使用 `*args` 和 `**kwargs` 来对传入的实参进行打包；

示例一：通过*args可以把实参打包成元组进行传递

```python
def print_multiple_args(*args):
    print(type(args))  # <class 'tuple'>
    print(args) # ('a', 'b', 'c')

# 传递方式一：
print_multiple_args('a', 'b', 'c') 
# 传递方式二：
print_multiple_args(*['a', 'b', 'c']) 
-----------------------------------------------
<class 'tuple'>
('a', 'b', 'c')
```

示例二：通过**kwargs可以把实参打包成字典进行传递

```python
def print_kwargs(**kwargs):
    print(type(kwargs))
    print(kwargs)

# 传递方式一：
print_kwargs(name='zgt', age=18)
# 传递方式二：
print_kwargs(**dict(name='zgt', age=18))
-----------------------------------------------
<class 'dict'>
{'name': 'zgt', 'age': 18}
```

示例三：混合参数注意传递顺序

```python
def print_all(a, *args, **kwargs):
    print(a)
    print(args)
    print(kwargs)

print_all('hello', 'world', name='zgt')
-----------------------------------------------
hello
('world',)
{'name': 'zgt'}
```



## 7 装饰器

关于装饰器的面试考察和知识点可以参考：[Python面试---装饰器](https://www.cnblogs.com/zzggtt/p/17099448.html)



## 8 == 和 is 的区别

`==` 运算符比较的是两个对象的值，而 `is` 运算符比较的是两个对象的标识(内存地址)；

```python
>>> a = 3.14
>>> b = 4.14
>>> a == b
False
>>> a is b
False
>>> a is None
False
```

`is`运算符比`==`速度快，因为它不能重载，所以Python不用寻找并调用特殊方法，而是直接对两个对象进行比较；

 而 `a == b` 是语法，等同于`a.__eq__(b)`，它继承自Object，用于比较两个对象存储的数据；



## 9 默认做浅拷贝

当我们需要列表（或多数内置的可变集合）进行复制时，最简单的操作方法是使用**内置的类型构造方法**或 `[:]` :

```python
>>> l1 = [1, [2, 3, 4], 9, (7, 8)]
>>> l2 = list(l1) # 复制列表l1
>>> l1 == l2  # 副本与原列表相等
True
>>> l1 is l2  #但是指代的不同的对象
False
```

上面例子中对包含了列表和元组的列表做了复制（浅复制），下面做些修改看看会有什么影响：

```python
>>> l1
[1, [2, 3, 4], 9, (7, 8)]
>>> l1.append(200)  # 把200追加到l1中对l2没有影响
>>> l1[1].remove(2) # 把内部l1[1]列表中对2删除,这对l2有影响
[1, [3, 4], 9, (7, 8), 200]
>>> l2
[1, [3, 4], 9, (7, 8)]
>>> l2[1] += [77, 88]  # 列表是可变对象, +=运算符就地修改列表
>>> l2[3] += (10, 11)  # 元组是不可变对象, +=运算符会创建一个新的元组
>>> l2
[1, [3, 4, 77, 88], 9, (7, 8, 10, 11)]
 >>> l1
[1, [3, 4, 77, 88], 9, (7, 8), 200]
```

可以看出把200追加到l1中对l2没有影响，但是把内部l1[1]列表中对2删除,这对l2有影响，因为l2[1] 和 l1[1] 绑定的列表是同一个，也就是说l2[1] 是l1[1] 的别名；

通过上面两个例子可以总结出：通过**内置的类型构造方法**或 `[:]`对可变类型进行复制操作时，默认是浅复制，即**副本共享内部可变类型对象的引用**。



那么有时候，如果需要的是深拷贝（完全复制一份副本，不共享可变类型的引用）时该如何操作呢？可以通过copy模块提供的deepcopy和copy函数可以为任意对象进行深浅拷贝操作：

```python
>>> l1 = [1, [2, 3, 4], 5, (6, 7, 8)]
>>> from copy import deepcopy
>>> l2 = deepcopy(l1)
>>> id(l1[1]), id(l2[1])  # 内部可变类型对象不是共享的引用,而是复制了一份
140172970968960, 140172971010304
```

但是深拷贝操作可能会存在循环引用的问题，在使用时需要注意：

```python
>>> a = [10, 20]
>>> b = [a, 30]
>>> a.append(b)
>>> a
[10, 20, [[...], 30]]
>>> c = deepcopy(a)
>>> c
[10, 20, [[...], 30]]
```



## 10 classmethod和staticmthod

在类中通过有三种方法，通过下面例子看下它们接受参数的区别：

```python
class Foo:
    def instance_method(*args):
        print(args)

    @classmethod
    def cls_method(*args):
        print(args)

    @staticmethod
    def static_method(*args):
        print(args)


f = Foo() # 实例化一个Foo对象
f.instance_method()  # 实例方法的参数是对象本身:(<__main__.Foo object at 0x7feba007d160>,)
f.cls_method()  # 类方法的参数是类本身:(<class '__main__.Foo'>,)
f.static_method() # 静态方法没有特殊的参数:()
```

其中实例方法是最常见的，它是用来定义**实例的操作方法**，接受特殊参数：`self`；

此外还有两个装饰器：classmethod 和 staticmethod；staticmethod很简单，它是定义在类中的普通方法，没有额外的特殊参数；而classmethod通常是用来定义**类的操作方法**，接受特殊参数`cls`，最常见的是定义备选的构造方法，例如：

```python
class Foo:
    ....
    @classmethod
    def cls_method(cls, *args):
        return cls(*args) # 返回一个新的实例
```



## 11 Python的私有属性和受保护的属性

Python不能像Java那样通过private 修饰符创建私有属性，举个例子：

```python
class Dog:
    mood = 'this is dog'

class Beagle(Dog):
    mood = 'this is beagle'
```

在上面的例子中，首先创建了一个Dog类，内部用到了mood属性（私有的没有将其开放），然后有创建了Beagle类继承自Dog类，但是在毫不知情的情况下又创建了mood属性，那么在继承的方法中就会把Dog类的mood属性覆盖；

Python有个简单的机制用于解决上面的兔子类意外覆盖“私有”属性，那就是在属性前加 `___` , 这样属性就变成了**私有属性**; 

Python会把属性名存入`__dict__` 属性中，而且会在前面加上一个下划线和类名，这个语言特性叫做“名称改写”例如:

```python
class Vector2d:
    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

v1 = Vector2d(3, 4)
print(v1.__dict__) # {'_Vector2d__x': 3.0, '_Vector2d__y': 4.0}
print(v1._Vector2d__x) # 3.0 虽然叫做私有属性，但是只要知道该写机制，任何人还是可以直接读取的
```



但是，不是所有的程序员都喜欢 `self.__x` 这种不对称的写法，他们约定了使用了一个下划线前缀编写**“受保护”属性**；Python解释器不会对使用单个下划线的属性名做特殊处理，不过这是很多Python程序员严格遵守的约定：不会在类外部访问这种属性，就像遵守使用全大写字母编写常量一样。

## 12 新式类和旧式类

（MRO）新式类的继承是根据C3算法，而旧式类是深度优先



## 13 new 和 init

在Python中通常把`__init__` 称为构造方法，这是从其他语言借鉴过来的；其实真正用于构建实例的是特殊方法`__new__`，这是个类方法（使用特殊方式处理，因此不必使用classmethod装饰器），它必须返回一个类的实例，回的实例会作为第一个参数（即self）传给`__init__`；所以`__init__`其实应该是初始化方法，真正的构造方法是`__new__`。

在日常使用中，其实很少用到new，在大型框架中经常会出现。

```python
class Person:
    def __new__(cls, *args, **kwargs): # 第一个参数为cls
        instance = super.__new__(cls, *args, **kwargs) # 调用父类的new方法生成实例
        return instance # 返回实例
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
```



## 14 元类

相比较于Java等静态语言，在Python一切皆对象的概念更加彻底，函数和类等等都是对象。

所以我们可以通过下面的方式去**生成类**：

```python
def create_class(class_name):
    if class_name == "User":
        class User:
            def __str__(self):
                return "create a user class"

        return User  # 把类当作对象返回

    elif class_name == "Company":
        class Company:
            def __str__(self):
                return "create a company class"

            
if __name__ == '__main__':
    my_class = create_class('User') # 通过函数返回一个类
    print(my_class) # <class '__main__.create_class.<locals>.User'>
    my_obj = my_class() # 通过返回的类创建实例
    print(my_obj) # create a user class
```

可以思考下，类这样的对象是由谁生成的呢，通过下面的示例可以得出答案：

```python
>>> a = 'hello'
>>> type(a)
<class 'str'>
>>> type(str)
<class 'type'> # "hello" <-- str <-- type
>>> class Student:
...     pass
...
>>> stu = Student
>>> type(stu)
<class 'type'>
>>> type(Student) 
<class 'type'> # stu <-- Student <-- type，
```

从示例中可以看出像内置的**str类**对象和自定义的Student类对象都是**由type类生成**的，所以type也称为元类。

type不仅可以打印对象类型，还可以**动态生成类**，下面是其不同的`__init__` 方法：

```python
def __init__(cls, what, bases=None, dict=None): # known special case of type.__init__
    """
        type(object_or_name, bases, dict)
        type(object) -> the object's type
        type(name, bases, dict) -> a new type
        # (copied from class doc)
        """
    pass
```

```python
>>> user = type('User', (), {"name": "zgt", 'age': 18})
>>> user.name
'zgt'
```

元类生成类对象还有另一种实现方式：即在声明类时指明**metaclass**参数，在类的声明中如果指定了metaclass，那么则会在**加载模块时**调用metaclass的`__new__`方法，它会**创建并返回一个类对象**：

```python
class UserMetaClass(type):
    """ 自定义元类,需要继承type """
    def __new__(cls, *args, **kwargs): 
        cls_instance = super().__new__(cls, *args, **kwargs)  # 调用父类type的new方法,返回一个类对象
        print(cls_instance) # <class '__main__.User'>
        return cls_instance


class User(metaclass=UserMetaClass): # 通过metaclass参数指明类的元类
    pass

----------------------------------------------
<class '__main__.User'>
```

在日常开发中，很少通过自定义元类来创建类对象；通常在框架中会看到自定义元类的身影，例如Django的ORM中就通过自定义元类把一些数据封装后再调用再创建类对象的。

------

那么，在指定了metaclass的情况下，通过类去实例化对象时又是怎样的调用过程呢？首先需要掌握一个知识：我们都知道类**实例化的对象**是**可调用对象**，前提是需要实现`__call__` 方法，例如：

```python
class Person:
    def __call__(self, *args, **kwargs):
        print("hello")

person = Person()
person()  # Person类的实例person + 可调用运算符() --> 调用__call__方法
```

再来看看，指定了metaclass的情况下又是如何的：

```python
class UserMetaClass(type):

    def __new__(cls, *args, **kwargs):
        print("in new")
        cls_instance = super().__new__(cls, *args, **kwargs)  # 调用父类type的new方法,返回一个类对象
        print(cls_instance)  # <class '__main__.User'>
        return cls_instance

    def __call__(self, *args, **kwargs):
        print("in call")


class User(metaclass=UserMetaClass):  # 通过metaclass参数指明类的元类
    def __init__(self, name, age):
        self.name = name
        self.age = age

# 元类的实例User类对象 + 调用运算符() --> 执行创建它的元类的__call__方法
user = User('zgt', 18)
print(type(user))  # <class 'NoneType'>
user.age # AttributeError: 'NoneType' object has no attribute 'age'
------------------------------------------------------
AttributeError: 'NoneType' object has no attribute 'age'
in new
<class '__main__.User'>
in call
<class 'NoneType'>
```

上面例子只是一个示例并不完整，因为元类的`__call__`方法并没有进行任何操作，下面是一个完整的示例：

```python
class UserMetaClass(type):

    def __new__(cls, *args, **kwargs):
        cls_instance = super().__new__(cls, *args, **kwargs)  # 调用父类type的new方法,返回一个类对象
        print(cls_instance)  # <class '__main__.User'>
        
        return cls_instance

    def __call__(self, *args, **kwargs):
        obj_instance = self.__new__(self)  # 
        self.__init__(obj_instance, *args, **kwargs) # 

        return obj_instance


class User(metaclass=UserMetaClass):  # 通过metaclass参数指明类的元类
    def __init__(self, name, age):
        self.name = name
        self.age = age


user = User('zgt', 18) # 会调用metaclass的__call__方法
print(type(user)) # <class '__main__.User'>
```



## 15 单例模式

>单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。通过单例模式可以保证系统中**一个类只有一个实例**而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。

单例模式的核心是：一个类只能创建一个实例；所以如果为了实现单例模式，需要在类创建实例的操作过程中进行一些必要的判断，来满足单例模式的要求。

-   通过 `__new__`实现：

    我们知道在类创建实例的过程中，首先会调用`__new__`方法生成并返回一个实例，再调用`__init__` 方法进行初始化操作，所以可以在生成实例之前做一个判断：

    ```python
    class Singleton:
        _instance = None
        def __new__(cls, *args, **kwargs):
            if cls._instance is not None:
                cls._instance = super().__new__(cls, *args, **kwargs)
            return cls._instance
    
    class User(Singleton):
        pass
    
    user1 = User()
    user2 = User()
    print(user1 is uers2)
    ```

-   通过元类实现：

    ```python
    class Singleton(type):
        def __init__(self, name, bases, attrs):
            self._instance = None
            super().__init__(self)
    
        def __call__(self, *args, **kwargs):
            if self._instance is not None:
                self._instance = super().__call__(self, *args, **kwargs)
            self.__init__(self._instance)
            return self._instance
    
    
    class User(metaclass=Singleton):
        pass
    
    
    user1 = User()
    user2 = User()
    print(user1 is user2)
    ```

-   共享属性：

    创建实例时把所有实例的`__dict__`指向同一个字典,这样它们具有相同的属性和方法.

    ```python
    class Borg(object):
        _state = {}
        def __new__(cls, *args, **kw):
            ob = super(Borg, cls).__new__(cls, *args, **kw)
            ob.__dict__ = cls._state
            return ob
    
    class MyClass2(Borg):
        a = 1
    ```

-   import导入：

    ```python
    # mysingleton.py
    class My_Singleton(object):
        def foo(self):
            pass
    
    my_singleton = My_Singleton()
    
    # to use
    from mysingleton import my_singleton
    
    my_singleton.foo()
    ```

-   装饰器：

    ```python
    def singleton(cls):
        instances = {}
        def getinstance(*args, **kw):
            if cls not in instances:
                instances[cls] = cls(*args, **kw)
            return instances[cls]
        return getinstance
    
    @singleton
    class MyClass:
      ...
    ```



## 16 可迭代对象&迭代器&生成器

参考我总结的博客：https://www.cnblogs.com/zzggtt/p/17105420.html











-   

-   

-   

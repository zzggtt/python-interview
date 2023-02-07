**Table of Contents**


   * [Python Interview](#python_interview)
      * [1 Python语言特性](#1-python语言特性)
      * [2 鸭子类型](#2-鸭子类型)
      * [3 猴子补丁](#3-猴子补丁)
      * [4 Python中对象自省](#4-Python对象自省)
      * [5 Python中函数参数传递](#5-Python中函数参数传递)

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


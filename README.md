**Table of Contents**


   * [Python Interview](#python_interview)
      * [1 Python语言特性](#1-python语言特性)

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



## 2 鸭子类型（duck typing）

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

## 3 猴子补丁（monkey patch）

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



## 4 Python对象的自省

自省（Introspection）是动态语言又一强大的特性，可以在运行时通过一定的机制查询到对象的内部结构；例如`dir()`、`isinstance()` 等。



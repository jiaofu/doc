python 特点
==
[TOC]
#高级特性
##切片
 **怎么取得列表的一部分数据**
 **怎么取末尾的一分部数据**
 **怎么取列表中的偶数呢**
 **怎么复制列表呢**
 用切片 [a:b:n]
 L = list(range(100))
 L[:10]
  L[-10:]
  L[10:20]
  L[:]

##迭代
Python中，迭代是通过for ... in来完成的，而很多语言比如C语言，迭代list是通过下标完成的
##列表生成式
**怎么用列表生成**
**列表生成次方**
**列表只生成偶数次方**
**两层循环**
rang(1,11)
 [x * x for x in range(1, 11)]
 [x * x for x in range(1, 11) if x % 2 == 0]
  [m + n for m in 'ABC' for n in 'XYZ']
##生成器
如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。
要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的[]改成()，就创建了一个generator：
generator保存的是算法，每次调用next(g)，就计算出g的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration的错误。

当然，上面这种不断调用next(g)实在是太变态了，正确的方法是使用for循环，因为generator也是可迭代对象：
##迭代器
可以直接作用于for循环的数据类型有以下几种：这些可以直接作用于for循环的对象统称为可迭代对象：Iterable。
一类是集合数据类型，如list、tuple、dict、set、str等；
一类是generator，包括生成器和带yield的generator function。
区别是而生成器不但可以作用于for循环，还可以被next()函数不断调用并返回下一个值，直到最后抛出StopIteration错误表示无法继续返回下一个值了。

可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator。
可以使用isinstance()判断一个对象是否是Iterator对象：
生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator。
把list、dict、str等Iterable变成Iterator可以使用iter()函数：

#面向对象编程
##类和实例
### 类和实例
####通过class关键字
class Student(object):
####创建实例
是通过类名+()实现的：  bart = Student()
####继承和多态
``` python
class Animal(object):
    def run(self):
        print('Animal is running...')

class Dog(Animal):
    pass
class Cat(Animal):
    pass

dog = Dog()
dog.run()

cat = Cat()
cat.run()
```
#### 多重集成
``` python
class Runnable(object):
    def run(self):
        print('Running...')

class Flyable(object):
    def fly(self):
        print('Flying...')

class Dog(Mammal, Runnable):
    pass

```
#### 定制类
##### __str__ 
打印类
#####  __repr__ 
程序开发者看到的字符串
#####  __iter__
如果一个类想被用于for ... in循环，类似list或tuple那样，就必须实现一个__iter__()方法，该方法返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的__next__()方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环。
```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环的条件
            raise StopIteration()
        return self.a # 返回下一个值
```
##### __getitem__
Fib实例虽然能作用于for循环，看起来和list有点像，但是，把它当成list来使用还是不行，比如，取第5个元素
``` python
Fib()[5]
```
##### __getattr__
正常情况下，当我们调用类的方法或属性时，如果不存在，就会报错.要避免这个错误，除了可以加上一个score属性外，Python还有另一个机制，那就是写一个__getattr__()方法，动态返回一个属性
``` python
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr=='score':
            return 99
```
##### __call__
一个对象实例可以有自己的属性和方法，当我们调用实例方法时，我们用instance.method()来调用。能不能直接在实例本身上调用呢？在Python中，答案是肯定的。

任何类，只需要定义一个__call__()方法，就可以直接对实例进行调用
``` python
class Student(object):
    def __init__(self, name):
        self.name = name

    def __call__(self):
        print('My name is %s.' % self.name)
        
     s = Student('Michael')
    s() # self参数不要传入
```
怎么判断一个变量是对象还是函数呢？其实，更多的时候，我们需要判断一个对象是否能被调用，能被调用的对象就是一个Callable对象，比如函数和我们上面定义的带有__call__()的类实例
``` python
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
#### 使用枚举
``` python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
value属性则是自动赋给成员的int常量，默认从1开始计数。
```
可以使用枚举的派生类
``` python
from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6

如果需要更精确地控制枚举类型，可以从Enum派生出自定义类
```
#### 使用元类
#### type 
type()函数既可以返回一个对象的类型，又可以创建出新的类型，比如，我们可以通过type()函数创建出Hello类，而无需通过class Hello(object)...的定义
``` python
>>> def fn(self, name='world'): # 先定义函数
...     print('Hello, %s.' % name)
...
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
```

要创建一个class对象，type()函数依次传入3个参数：

    class的名称；
    继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
    class的方法名称与函数绑定，这里我们把函数fn绑定到方法名hello上。

通过type()函数创建的类和直接写class是完全一样的，因为Python解释器遇到class定义时，仅仅是扫描一下class定义的语法，然后调用type()函数创建出class。

正常情况下，我们都用class Xxx...来定义类，但是，type()函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂。

#### metaclass
控制类的创建行为，还可以使用metaclass。
先定义metaclass，就可以创建类，最后创建实例。
``` python
定义ListMetaclass，按照默认习惯，metaclass的类名总是以Metaclass结尾，以便清楚地表示这是一个metaclass：
# metaclass是类的模板，所以必须从`type`类型派生：
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)
```
当我们传入关键字参数metaclass时，魔术就生效了，它指示Python解释器在创建MyList时，要通过ListMetaclass.__new__()来创建，在此，我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。

__new__()方法接收到的参数依次是：

    当前准备创建的类的对象；
    
    类的名字；
    
    类继承的父类集合；
    
    类的方法集合。

有了ListMetaclass，我们在定义类的时候还要指示使用ListMetaclass来定制类，传入关键字参数metaclass
``` python
class MyList(list, metaclass=ListMetaclass):
    pass

```
### 属性

#### 类属性
``` python
class Student(object):
    name = 'Student'
```
定义了一个类属性后，这个属性虽然归类所有，但类的所有实例都可以访问到
Student.name 可以
实例.name 可以

#### 访问数据限制
内部属性不被外部访问，可以把属性的名称前加上两个下划线__
``` python
class Student(object):

    def __init__(self, name, score):
        self.__name = name
        self.__score = score

    def print_score(self):
        print('%s: %s' % (self.__name, self.__score))

bart = Student('Bart Simpson', 59)
bart.__name
Traceback (most recent call last):
```
#### 限制添加属性 使用__slots__
``` python
class Student(object):
    __slots__ = ('name', 'age')
```
#### 限制属性的范围 @property
Python内置的@property装饰器就是负责把一个方法变成属性调用的
``` python
class Student(object):
    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value
        
    @property
    def age(self):
        return 2015 - self._birth
```
神奇的@property，我们在对实例属性操作的时候，就知道该属性很可能不是直接暴露的，而是通过getter和setter方法来实现的。
还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性：
### 方法

####构造器呢?  
**def __init__(self, name, score):** __init__方法的第一个参数永远是self，表示创建的实例本身，因此，在__init__方法内部，就可以把各种属性绑定到self，因为self就指向创建的实例本身。
有了__init__方法，在创建实例的时候，就不能传入空的参数了，必须传入与__init__方法匹配的参数，但self不需要传
#### 定义方法
定义一个方法，除了第一个参数是self外，其他和普通函数一样。要调用一个方法，只需要在实例变量上直接调用，除了self不用传递，其他参数正常传入：
```python
 def print_score(self):
        print('%s: %s' % (self.name, self.score))
```
##获取对象信息
### type()
它返回对应的Class类型 
type(123)
type(abs)
### isinstance()
class的继承关系来说，使用type()就很不方便。我们要判断class的类型，可以使用isinstance()函数
### dir()
如果要获得一个对象的所有属性和方法，可以使用dir()函数，它返回一个包含字符串的list，比如，获得一个str对象的所有属性和方法：
#### 配合方法 getattr()、setattr()以及hasattr()
##实例属性和类属性
#面向对象高级编程
#错误
#正则表达式




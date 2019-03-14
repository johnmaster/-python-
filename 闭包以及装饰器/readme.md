# 闭包 
1.闭包介绍<br>
<tab>闭包概念：在一个内部函数中，对外部作用域中的变量进行引用，（并且一般外部函数的返回值为内部函数），那么内部函数就被认为是闭包。
举一个例子：<br>
```python
    def a(x):
        def b(y):
            return x+y
        return b
```
在函数a中定义了一个b函数，b函数访问了外部函数a的变量，并且函数返回值为b函数。<br>
```python
a_1 = a(1)
print('function:',a_1)
print('result:',a_1(1))
```
上述的代码*a_1*是一个函数，上面代码的执行结果<br>
function: <function a.<locals>.b at 0x000002382A3BAAE8><br>
result: 2<br>
从结果我们不难看出，a_1是函数b,而不是函数a，这是因为return回来的是函数b。
2.闭包可以保存运行环境
```python
   _list = []
   for i in range(3):
       def func(a):
           return i+a
       _list.append(func)
   
   for f in _list:
       print(f(1))
```
上面的函数输出不是1，2，3，而是3，3，3。<br>
这是因为在Python中，循环体内定义的函数是无法保存循环执行过程中的不停变化的外部变量的，即普通函数无法保存运行环境！这种事情应该让闭包来做。
```python
_list = []
for i in range(3):
    def func(i):
        def f_closure(a):
            return i+a
        return f_closure
    _list.append(func(i))

for i in _list:
    print(f(1))
```
<br>
# 装饰器
装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。概括的讲，装饰器的作用就是为
已经存在的对象增加额外的功能。<br>
先看一个简单的例子：<br>
```python
def foo():
  print('i am foo ')
```
现在有一个新的需求，希望在程序执行的过程中可以打印出来一条信息*foo is running*于是在代码中添加这行语句<br>
  ```python
  def foo():
      print('i am foo')
      print("foo is running")
  ```
但是此时有很多和函数有这样的需求，这样该怎么做？再写一个print("foo is running")放在foo()以及很多函数中去么？这样就造成了大量雷同的代码，为了减少重复
写代码，我们可以这样做，重新定义一个函数，专门处理类似print("foo is running")这样的命令。<br>
```python
def use_logging(func):
    print("%s is running " % func.__name__)
    func()

def bar():
    print('i am bar')

use_logging(bar)
```
这种方式从逻辑上不难理解，但是这样的话，我们每次都要将一个函数作为参数传递给*use_logging*函数，而且这种方式已经
破坏了原有的代码逻辑结构，之间执行业务逻辑时，执行运行bar(),但是现在不得不改成use_logging(bar)。对于这种方式，
我们可以使用**装饰器**来应对。<br>
**简单装饰器**<br>
```python
def use_logging(func):
    def wrapper(*args, **kwargs):
        print("%s is running" % func.__name__)
        return func(*args, **kwargs)
    return wrapper

def bar():
    print("i am bar")

bar = use_logging(bar)
bar()
```
函数use_logging就是装饰器，它把真正执行业务方法的func包裹再函数里面，看起来像bar被use_logging装饰了。<br>
@符号是装饰器的语法糖，在定义函数的时候使用，避免再一次赋值操作。<br>
```python
def use_logging(func):
    def wrapper(*args, **kwargs):
        print("%s is running" % func.__name__)
        return func(*args,**kwargs)
    return wrapper

@use_logging
def bar():
    print("i am bar")

@use_logging
def foo():
    print("i am foo")

bar()
```
如上所示，这样我们就可以省去bar = use_logging(bar)这一句了，直接调用bar()即可得到想要的结果。如果我们有其他的类似函数，我们
可以继续调用装饰器来修饰函数，而不用重复修改函数或者增加新的封装，这样，我们就提高了程序的可重复利用性，并增加了程序的可读性。
<br>
**带参数的装饰器**<br>
装饰器还有更大的灵活性，例如带参数的装饰器：在上面的装饰器调用中，比如*@use_logging*,该装饰器唯一的参数就是执行业务的函数。
装饰器的语法允许我们在调用时，提供其他的参数，比如@decorator(a),这样，就为装饰器的编写和使用提供了更大的灵活性。<br>
```python
def use_logging(level):
    def decorator(func):
        def wrapper(*args, **kwargs):
            if level == "warn":
                print("%s is running " % func.__name__)
            return func(*args)
        return wrapper
    return decorator

@use_logging(level="warn")
def foo(name="foo"):
    print("i am %s " % name )

foo()
```
上面的use_logging是允许带参数的装饰器。它实际上是对原有装饰器的一个函数封装，并返回一个装饰器。
这个效果相当于 foo = use_logging(level="warn")(foo(name="foo"))<br>
**类装饰器**<br>
相比于函数装饰器，类装饰器具有灵活度大，高内聚，封装性等优点。使用类装饰器还可以依靠类内部的\_\_call\_\_方法，
当使用@形式将装饰器附加到函数上时，就会调用这个方法。<br>
```python
class Foo(object):
    def __init__(self, func):
        self._func = func
    def __call__(self):
        print('class decorator running')
        self._func()
        print('class decorator ending')

@Foo
def bar():
    print('It is bar')

bar()
```
**functools.wraps**<br>
使用装饰器极大的复用了代码，但是装饰器有一个缺点就是原函数的元信息不见了，函数也是对象，所以函数也有\_\_name\_\_
等属性，但是你们去看一下经过装饰器装饰后的函数，它们的\_\_name\_\_已经改变了。<br>
```python
def logged(func):
    def with_logging(*agrs, **kwargs):
        print(func.__name__ + " was called ")
        return func(*args, **kwargs)
    return with_logging

@logged
def f(x):
    """does some math """
    print("current the number is %d " % x)

print(f.__name__)
print(f.__doc__)
```
可以看得到f.\_\_name\_\_输出的是with\_logging,f.\_\_doc\_\_什么也没输出。<br>
为了应对这个问题，我们使用functools.wraps,wraps本身也是一个装饰器，它能把原函数的元信息拷贝到装饰器函数中，这使得
装饰器函数也有和原函数一样的元信息了。<br>
```python
from functools import wraps
def logged(func):
    @wraps(func)
    def with_logging(*agrs, **kwargs):
        print(func.__name__ + " was called ")
        return func(*args, **kwargs)
    return with_logging

@logged
def f(x):
    """does some math """
    print("current the number is %d " % x)

print(f.__name__)
print(f.__doc__)
```

















# 闭包 
1.闭包介绍<br>
<tab>闭包概念：在一个内部函数中，对外部作用域中的变量进行引用，（并且一般外部函数的返回值为内部函数），那么内部函数就被认为是闭包。
举一个例子：<br>
&#8195;def a(x):<br>
&#8195;&#8195;def b(y):<br>
&#8195;&#8195;&#8195;return x+y<br>
&#8195;&#8195;return b<br>
在函数a中定义了一个b函数，b函数访问了外部函数a的变量，并且函数返回值为b函数。<br>
a_1 = a(1)<br>
print('function:',a_1)<br>
print('result:',a_1(1))<br>
上述的代码*a_1*是一个函数，上面代码的执行结果<br>
function: <function a.<locals>.b at 0x000002382A3BAAE8><br>
result: 2<br>
从结果我们不难看出，a_1是函数b,而不是函数a，这是因为return回来的是函数b。
# 装饰器
装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。概括的讲，装饰器的作用就是为
已经存在的对象增加额外的功能。<br>
先看一个简单的例子：<br>
def foo():<br>
&#8195;print('i am foo ')<br>
现在有一个新的需求，希望在程序执行的过程中可以打印出来一条信息*foo is running*于是在代码中添加这行语句<br>
def foo():<br>
&#8195;print('i am foo')<br>
&#8195;print("foo is running")<br>
但是此时有很多和函数有这样的需求，这样该怎么做？再写一个print("foo is running")放在foo()以及很多函数中去么？这样就造成了大量雷同的代码，为了减少重复
写代码，我们可以这样做，重新定义一个函数，专门处理类似print("foo is running")这样的命令。<br>
def use_logging(func):<br>
&#8195;print("%s is running " % func.__name__)<br>
&#8195;func()<br>
<br>
def bar():<br>
&#8195;print('i am bar')<br>
<br>
use_logging(bar)<br>
这种方式从逻辑上不难理解，但是这样的话，我们每次都要将一个函数作为参数传递给*use_logging*函数，而且这种方式已经
破坏了原有的代码逻辑结构，之间执行业务逻辑时，执行运行bar(),但是现在不得不改成use_logging(bar)。对于这种方式，
我们可以使用**装饰器**来应对。<br>
**简单装饰器**<br>
def use_logging(func):<br>
&#8195;def wrapper(*args, **kwargs):<br>
&#8195;&#8195;print("%s is running" % func.__name__)<br>
&#8195;&#8195;return func(*args, **kwargs)<br>
&#8195;return wrapper<br>
<br>
def bar():<br>
&#8195;print("i am bar")<br>
<br>
bar = use_logging(bar)<br>
bar()<br>
<br>
函数use_logging就是装饰器，它把真正执行业务方法的func包裹再函数里面，看起来像bar被use_logging装饰了。<br>
@符号是装饰器的语法糖，在定义函数的时候使用，避免再一次赋值操作。<br>
def use_logging(func):<br>
&#8195;def wrapper(*args, **kwargs):<br>
&#8195;&#8195;print("%s is running" % func.__name__)<br>
&#8195;&#8195;return func(*args, **kwargs)<br>
&#8195;return wrapper<br>
<br>
@use_logging<br>
def bar():<br>
&#8195;print("i am bar")<br>
<br>
@use_logging<br>
def foo():<br>
&#8195;print("i am foo")<br>
<br>
bar()<br>
<br>
如上所示，这样我们就可以省去bar = use_logging(bar)这一句了，直接调用bar()即可得到想要的结果。如果我们有其他的类似函数，我们
可以继续调用装饰器来修饰函数，而不用重复修改函数或者增加新的封装，这样，我们就提高了程序的可重复利用性，并增加了程序的可读性。
<br>
**带参数的装饰器**<br>
装饰器还有更大的灵活性，例如带参数的装饰器：在上面的装饰器调用中，比如*@use_logging*,该装饰器唯一的参数就是执行业务的函数。
装饰器的语法允许我们在调用时，提供其他的参数，比如@decorator(a),这样，就为装饰器的编写和使用提供了更大的灵活性。<br>
def use_logging(level):<br>
&#8195;def decorator(func):<br>
  &#8195;&#8195;def wrapper(*args, **kwargs):<br>
  &#8195;&#8195;&#8195;if level == "warn":<br>
  &#8195;&#8195;&#8195;&#8195;print("%s is running " % func.__name__)<br>
  &#8195;&#8195;&#8195;return func(*args)<br>
  &#8195;&#8195;return wrapper<br>
  &#8195;return decorator<br>
  @use_logging(level="warn")<br>
  def foo(name="foo"):<br>
  print("i am %s " % name )<br>
  foo<br>

















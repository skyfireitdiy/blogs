# python 偏函数

## 从进制转换说起

假设我们有一个程序，要把字符串转成整数：

```python
data = "12345"
n = int(data)
```

这时n就是整数12345.

后来，我们发现"12345"其实是16进制的数字。查阅文档，我们发现，其实int()还有一个参数：base

```text
class int(object)
 |  int([x]) -> integer
 |  int(x, base=10) -> integer
 |  
 |  Convert a number or string to an integer, or return 0 if no arguments
 |  are given.  If x is a number, return x.__int__().  For floating point
 |  numbers, this truncates towards zero.
 |  
 |  If x is not a number or if base is given, then x must be a string,
 |  bytes, or bytearray instance representing an integer literal in the
 |  given base.  The literal can be preceded by '+' or '-' and be surrounded
 |  by whitespace.  The base defaults to 10.  Valid bases are 0 and 2-36.
 |  Base 0 means to interpret the base from the string as an integer literal.
 |  >>> int('0b100', base=0)
 |  4
 ......
```

于是，我们将程序改成这样：

```python
data = "12345"
n = int(data, base=16)
```

这时候，我们的n就是74565 (0x12345对应的10进制数字)

我们的代码中要写很多这样的转换，所以，我们可能要写很多的`base=16`,所以，我们将转换16进制这个功能封装成一个函数：

```python
def my_int(s , base=16):
    return int(s, base)

data = "12345"
n = my_int(data)
```
后续只需要调用my_int函数，默认的base就是16了，也可以手动设置base的值。

## 使用functools

上面的功能，在一个内置包functools中已经有现成的函数了。

使用functools的代码如下:

```python
import functools

my_int = functools.partial(int, base=16)

data = "12345"
n = my_int(data)
```

上面的代码，使用functools.partial将int的base参数绑定为16。

查看functools.partial的帮助：

```text
class partial(builtins.object)
 |  partial(func, *args, **keywords) - new function with partial application
 |  of the given arguments and keywords.
 |  
 |  Methods defined here:
 |  
 |  __call__(self, /, *args, **kwargs)
 |      Call self as a function.
 |  
 |  __delattr__(self, name, /)
 |      Implement delattr(self, name).
 |  
 |  __getattribute__(self, name, /)
 |      Return getattr(self, name).
 |  
 |  __reduce__(...)
 |      Helper for pickle.
 |  
 |  __repr__(self, /)
 |      Return repr(self).
 |  
 |  __setattr__(self, name, value, /)
 |      Implement setattr(self, name, value).
 |  
 |  __setstate__(...)
 |  
 |  ----------------------------------------------------------------------
 |  Static methods defined here:
 |  
 |  __new__(*args, **kwargs) from builtins.type
 |      Create and return a new object.  See help(type) for accurate signature.
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |  
 |  args
 |      tuple of arguments to future partial calls
 |  
 |  func
 |      function object to use in future partial calls
 |  
 |  keywords
 |      dictionary of keyword arguments to future partial calls
```

我们发现，functools.partial其实是一个类，只是他实现了__call__成员，使自身表现的像个函数。

## 使用参数名

假设我们有个除法函数：

```python
def func(a,b):
    return a/b
```

我们需要固定被除数的值，即：

```python
def my_func1(a=10, b):
    return func(a,b)
```

使用functools可以这样写：

```python
my_func1 = functools.partial(func, 10)
```

但是，如果我们要固定除数：

```python
def my_func2(a, b=10):
    return func(a/b)
```

使用functools该怎么写呢，这时候就需要参数名了：

```python
my_func2 = functools.partial(func, b=10)
```

这是，函数的参数b被固定，传入的参数会被当做参数a：

```python
print(my_func2(20))
```

输出：

```text
2.0
```

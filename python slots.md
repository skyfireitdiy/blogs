# python `__slots__`

python 作为一个动态语言，在运行过程中为对象添加属性是他的特性之一，如下：

```python
class A:
    ...


a = A()
a.name = "Hello"
a.age = 25
print(a.name)
print(a.age)
```

程序输出：

```text
Hello
25
```

有的时候，我们需要限制属性的添加，比如我们有个Human的类，只允许用户添加name和age属性，其他属性不允许添加：

```python
class Human(object):
    __slots__ = ('name', 'age')


a = Human()
a.name = "Hello"
a.age = 25
print(a.name)
print(a.age)

a.tall = 175
print(a.tall)
```

在运行到`a.tall = 175`的时候，会抛出以下异常：

```text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Human' object has no attribute 'tall'
```

除了`name`和`age`，其他属性都被限制了。
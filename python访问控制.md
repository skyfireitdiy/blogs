# python 访问控制

## public

python 类中正常命名的变量都是可以通过外部访问的。比如：

```python
class A:
    def __init__(self):
        self.a = 10
        self.b = "hello"

t = A()
t.a = 20
t.b="world"
```

这样都是OK的。

## private

**使用双下划线开头但不以双下划线结尾的的变量是private的，不能被外部访问，只能被类成员访问。**
```python
class B:
    def __init__(self):
        self.__a = 10
        self.__b = "hello"
    def disply(self):
        print(self.__a)
        print(self.__b)
        

t = B()
t.disply()
```

可以正常使用，但是如果这样：

```python
print(t.__a)
```

就会出错：

```text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'B' object has no attribute '__a'
```

我们可以使用dir看一下t的属性：

```python
dir(t)
```
输出：

```text
['_B__a', '_B__b', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__l
e__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'disply']
```

可以看到__a和__b被重命名为了_B__a和_B__b，在成员名称前面添加了_B类名，所以我们依然可以通过 类名+成员变量名 访问：

```python
print(t._B__a)
```

只是一般要避免这种做法。

## 例外

上文提到`使用双下划线开头但不以双下划线结尾的的变量是private的`，所以如果变量使用双下划线开头，以双下滑线结尾，就不再是private的了。

```python
class C:
    def __init__(self):
        self.__a__ = 10
        self.__b__ = "hello"

t = C()
print(t.__a__)
print(t.__b__)

```

这种用法一般是python内置的用法，开发中一般不建议使用。
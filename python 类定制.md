# python 类定制

python 中的形如__xxx__的函数或属性都是有特殊含义的。

## `__len__`

__len__是为了可以让len函数调用：

```python
class A:
    def __len__(self):
        return 20


a = A()
print(len(a))
```

输出：

```text
20
```

## `__str__`

__str__是为了让print打印的信息好看一些：

```python
class A:
    def __str__(self):
        return "this is class A"


a = A()
print(a)
```

输出：

```text
this is class A
```

## `__repr__`

__repr__是为了在调试的时输出的信息可读，在终端输入一个对象的时候，会直接打印对象的值，设置了__repr__就会打印指定的值。

在python交互式终端输入以下代码：


```python
class A:
    def __repr__(self):
        return "debug info"


a = A()
a
```

输出

```text
debug info
```

## `__iter__`

__iter__返回一个迭代器，供for使用：

```python
class A:
    def __iter__(self):
        return (i for i in range(10))


a = A()
for i in a:
    print(i)
```

输出：

```text
0
1
2
3
4
5
6
7
8
9
```

## `__next__`

使用`__next__`可以让对象自身变成一个可迭代对象

```python
class A:
    def __init__(self):
        self.__data = 0
    def __iter__(self):
        return self
    def __next__(self):
        self.__data += 1
        if self.__data > 5:
            raise StopIteration()
        return self.__data


a = A()
for i in a:
    print(i)
```

输出

```text
1
2
3
4
5
```

## `__getitem__`

__getitem__可以让你的对象支持像列表或者字典那样使用下标操作。

```python
class A:
    def __getitem__(self, index):
        if isinstance(index, int):
            return index*2
        if isinstance(index, slice):
            if index.step is not None:
                return [i*2 for i in range(index.start, index.stop, index.step)]
            else:
                return [i*2 for i in range(index.start, index.stop)]
        

a = A()
print(a[5])
print(a[10:15])
print(a[2:10:2])
```

输出：

```text
10
[20, 22, 24, 26, 28]
[4, 8, 12, 16]
```

## `__getattr__`

__getattr__在访问不存在的属性的时候会被调用：

```python
class A:
    def __init__(self):
        self.a = 5
    def __getattr__(self, name):
        print("Get", name)
        return 0


a = A()
print(a.a)
print(a.b)
```

程序输出：

```text
5
Get b
0
```

## `__call__`

__call__可以让对象和函数一样可以被调用：

```python
class A:
    def __call__(self, a , b):
        return a+b


a = A()
print(a(5,10))
```

输出：

```text
15
```
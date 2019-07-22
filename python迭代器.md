# python 可迭代对象(Iterable)与迭代器(Iterator)

## 可迭代对象

可迭代对象可以简单理解为可以再for语句中使用的对象：

如：

```python
arr = [1,2,3,4,5]
for i in arr:
    print(i)
```

其中arr可以在for中被使用，所以arr是可迭代对象。

再如：

```python
data = (i*2 for i in range(100))
for i in data:
    print(i)
```

data可以在for中使用，所有data是可迭代对象。可以看出，所有的生成器都是可迭代对象。

可以有一个简单的办法判断一个对象是否是可迭代对象：

```python
from collections import Iterable
isinstance([], Iterable)
isinstance((), Iterable)
isinstance({}, Iterable)
isinstance("", Iterable)
isinstance(1.2, Iterable)
```

依次输出：

```text
True
True
True
True
False
```

## 迭代器

迭代器的作用是为了执行惰性计算，就是只有在某个值被使用时才会计算它的值。所以，所有的生成器都是迭代器。

迭代器的一个特性是可以被next函数调用,这是next函数的文档，所以，可以被next调用的对象就是迭代器。

```text
next(...)
    next(iterator[, default])
    
    Return the next item from the iterator. If default is given and the iterator
    is exhausted, it is returned instead of raising StopIteration.
```

判断迭代器的方法和判断可迭代对象类似：

```python
from collections import Iterator
isinstance([], Iterator)
isinstance((), Iterator)
isinstance({}, Iterator)
isinstance("", Iterator)
isinstance(1.2, Iterator)
isinstance((i for i in []), Iterator)
```

依次输出：

```text
False
False
False
False
False
True
```

可以看到，dict、list、str等对象是可迭代对象，可以被在for中使用，但是不是迭代器，不能使用next函数。

## 区别

可迭代对象与迭代器的区别在于：

- 可迭代对象，可以使用循环访问对象中的元素，对象中的元素可以是调用的时候计算出来，也可以是预先计算出来，调用的时候直接获取。
- 迭代器，只有在访问的时候才会知道下一个值是什么，在使用的时候才会计算新值。除此外，它包含可迭代对象的所有特性。
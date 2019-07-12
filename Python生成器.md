# python生成器

生成器主要应用于内存紧缺的情况，比如需要一个100万项的数列，直接存入内存是很浪费的，如果可以在使用的时候实时计算出来，那就再好不过了。

## 列表生成式版本

```python
[x for x in range(1000000)]
```

这样会直接生成一个列表，存储在内存中。

## 生成器

只需要将[]改为()，列表生成式就成了一个生成器。

```python
(x for x in range(1000000))
```

使用的时候，可以这样：

```python
g = (x for x in range(1000000))
print(next(g))
print(next(g))
print(next(g))
print(next(g))
```
程序会依次输出

```text
0
1
2
3
```

每次调用next，都会从上一次的地方继续计算。

可以像列表一样迭代：

```python
g = (x for x in range(10))
for i in g:
    print(i)
```

输出：

```
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

## 使用yield

列表生成式形式的生成器作用有限，不能完成复杂的逻辑，比如，我们需要一个斐波那契数列的生成器，使用列表生成器形式的就很难完成。我们可以使用yeild完成这项任务：

```python
def fib():
    a = 1
    b = 1
    while True:
        yield a
        a, b = b, a+b
```

带有yield的函数返回一个生成器：

```python
g = fib()
print(next(g))
print(next(g))
print(next(g))
print(next(g))
print(next(g))
print(next(g))
```

输出：

```text
1
1
2
3
5
8
```

在调用next的时候，会进入fib函数，执行到yield的时候返回一个值。下一次再次调用next的时候，fib函数会从yeild处继续执行，直到下一个yield。

仔细观察不难发现fib函数可以生成的斐波那契数列是无限长的，而这个功能使用列表生成器是无法做到的。


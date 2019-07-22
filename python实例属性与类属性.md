# python 实例属性与类属性

## 实例属性

实例属性就是每个实例相互独立的属性：

```python
class A:
    def __init__(self):
        self.a = 10

a = A()
print(a.a)
b = A()
print(b.a)
a.a = 50
print(a.a)
print(b.a)
```

程序输出：

```text
10
10
50
10
```

修改一个实例的属性，不会影响其他实例。

## 类属性

类属性是类自身的属性，与每个实例都没关系。

```python
class A:
    a = 10


a = A()
print(a.a)
A.a = 50
b = A()
print(b.a)
```

输出：

```text
10
50
```

## 联系

类属性会被复制到实例属性，但是复制实例属性一旦重新赋值，就与类属性分离了。表现如下：

```python
class A:
    a = 10


a = A()
b = A()
print(A.a)  # 10
print(a.a)  # 10 因为被复制到self.a
print(b.a)  # 10
A.a = 100
print(A.a)  # 100
print(a.a)  # 100 没有重新赋值，所以这里的a还是A.a
print(b.a)  # 100
a.a = 1000
print(A.a)  # 100
print(a.a)  # 1000 重新赋值，所以这里的a与A.a没有联系
print(b.a)  # 100 
A.a = 10000
print(A.a)  # 10000
print(a.a)  # 1000 独立的a
print(b.a)  # 10000 b.a在赋值前保持与A.a的关联
```
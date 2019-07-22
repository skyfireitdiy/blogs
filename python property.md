# python property

在定义类时，可以直接把成员暴露出去，外部代码直接访问修改：

```python
class Student:
    def __init__(self):
        self.name = ""
        self.age = 26
    def show(self):
        print("name:", self.name, "age:", self.age)


s = Student()
s.name = "skyfire"
s.age = 25
s.show()
```

程序输出：

```text
name: skyfire age: 25
```

但是，我们也可以这么写代码：

```python
ss = Student()
ss.name = "skyfire"
ss.age = "不是数字的年龄"
ss.show()
```

输出：

```text
name: skyfire age: 不是数字的年龄
```

也就是说，我们无法对赋值做限制。所以我们需要写一个setter：

```python
class Student:
    def __init__(self):
        self.name = ""
        self.age = 26
    def set_age(self, a):
        if not isinstance(a , int):
            print("type error")
            return
        self.age = a
    def show(self):
        print("name:", self.name, "age:", self.age)


s = Student()
s.name = "skyfire"
s.set_age(25)
s.show()
s.set_age("不是数字")
s.show()
```

但是这样，每次赋值都需要调用set_age函数（而且也无法保证用户一定会调用set_age而不是直接赋值，毕竟age是public的）


所以，就有一种比较理想的办法解决这个问题：

```python
class Student:
    def __init__(self):
        self.name = ""
        self.__age = 26
    @property
    def age(self):
        return self.__age
    @age.setter
    def age(self, a):
        if not isinstance(a , int):
            print("type error")
            return
        self.__age = a
    def show(self):
        print("name:", self.name, "age:", self.age)


s = Student()
s.name = "skyfire"
s.age = 25
s.show()
s.age = "不是数字"
s.show()
```

以上代码做了以下几件事：

1. 将age变为__age，这是为了防止与函数同名，并且设置为私有，用户不能直接访问
2. 增加setter函数和getter函数，但是两个函数都叫age
3. 给setter函数使用@age.setter装饰器，给getter函数使用@property装饰器

这样一来，使用的时候就像使用实例的public属性一样，但是又会进入getter和setter函数做类型检查等工作
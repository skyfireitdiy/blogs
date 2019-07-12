# python继承与多态

## 继承

python 的类是可以继承的：

```python
class Animal:
    def run(self):
        print("animal running ...")


class Dog(Animal):
    ...


d = Dog()
d.run()
```

输出

```text
animal running ...
```

python 的继承与其他语言并无太大区别，所以本片主要讲多态。

## 多态

由于 python 是动态类型的语言，所以只需要提供相同的接口，就可以有相同的用法，如下：

```python
class Animal:
    def run(self):
        print("animal running ...")


class Dog:
    def run(self):
        print("dog running ...")


def run(ani):
    ani.run()

a = Animal()
b = Dog()
run(a)
run(b)
```

程序输出：

```text
animal running ...
dog running ...
```

我们注意一下，上面的 Animal 和 Dog 类之间并没有继承关系，他们只是提供了相同的接口。如果Dog 没有提供 run 接口，run函数就无法调用它。

这里的run函数，只要某对象提供了run方法，就可以作为参数进行调用。所以我们无法对类型做限制，因为参数的类型是无法预料的。

如果我们需要在调用前检查一下类型，这时候就需要多态了。

```python
class Animal:
    def run(self):
        print("animal running ...")


class Dog(Animal):
    def run(self):
        print("dog running ...")


class Stone():
    def run(self):
        print("stone running ...")


class Fish(Animal):
    ...


def run(ani):
    if isinstance(ani, Animal):
        ani.run()
    else:
        print("type error")

a = Animal()
b = Dog()
c = Stone()
d = Fish()
run(a)
run(b)
run(c)
run(d)
```

程序输出：

```text
animal running ...
dog running ...
type error
animal running ...
```

上面的程序中，Stone不是继承自Animal，所以在类型检查会被过滤掉，Fish继承自Animal，所以当Fish类没有实现run方法的时候，父类的run会被调用。


# `Python`注解

使用过`flask`框架的程序员一定见过如下的`python`代码：

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def func():
    return "Welcome to Flask!"

app.run(port=8080)
```

但是相信有很多人对其中的`@app.route("/")`存在一些疑惑，现在我们就讲解一下这个传说中的“注解”。

我们先从一个函数包装讲起。假设我们有一个程序：

```python
def print_hello(name):
    print("hello",name)

print_hello("skyfire")
```

直接运行这个程序会打印：

```text
hello skyfire

```

假设我们想在函数调用前后做一些事情，于是我们将程序做了一些改造：

```python
def print_hello(name):
    print("hello",name)

def wrapper(name):
    print("before print_hello")
    print_hello(name)
    print("after print_hello")

wrapper("skyfire")
```

这时候程序输出：

```text
before print_hello
hello skyfire
after print_hello
```

以上程序添加了一个新的函数`wrapper`，在这个函数中调用`print_hello`，并且在调用前后执行一些附加的代码。然后调用`wrapper`代替原来的`print_hello`。

随着程序功能不断完善～～～，我们又多了一些函数，然后又给这些函数增加了“`wrapper`”:

```python
def print_hello(name):
    print("hello",name)

def shake_hand(name):
    print("shake to", name)

def wrapper_print_hello(name):
    print("before print_hello")
    print_hello(name)
    print("after print_hello")


def wrapper_shake_hand(name):
    print("before shake_hand")
    shake_hand(name)
    print("after shake_hand")

wrapper_print_hello("skyfire")
wrapper_shake_hand("skyfire")
```

函数一个个增加，`wrapper`数量也一个个增加，代码膨胀严重。于是，我们想创建一个通用的`wrapper`来包装每个函数：

```python
def print_hello(name):
    print("hello",name)

def shake_hand(name):
    print("shake to", name)

def wrapper(fn, name):
    print("before", fn.__name__)
    fn(name)
    print("after",fn.__name__)

wrapper(print_hello,"skyfire")
wrapper(shake_hand,"skyfire")
```

这样一来，我们的`wrapper`更通用了一些。

很久很久以后，突然出现了一个这样的函数：

```python
def borrow(name,thing):
    print(name,"borrow",thing)
```

`wrapper`不得不升级了,于是有了以下的代码：

```python
def print_hello(name):
    print("hello",name)

def shake_hand(name):
    print("shake to", name)

def borrow(name,thing):
    print(name,"borrow",thing)

def wrapper(fn, *args, **kwds):
    print("before", fn.__name__)
    fn(*args,**kwds)
    print("after",fn.__name__)


wrapper(print_hello,"skyfire")
wrapper(shake_hand,"skyfire")
wrapper(borrow, "skyfire", "pen")
```

这下问题又解决了。但是，需求总是在不断更新的，我们想在`borrow`上再增加一个包装，来打印一下参数。此时这种办法就行不通了，我们需要一个包装好的函数，于是，我们修改了一下我们的`wrapper`（为了简洁，去除了多余的代码）:

```python
def borrow(name,thing):
    print(name,"borrow",thing)

def wrapper(fn):
    def child(*args,**kwds):
        print("before", fn.__name__)
        fn(*args,**kwds)
        print("after",fn.__name__)
    return child

wrapper(borrow)("skyfire", "pen")
```

这次的修改比较大，我们将原来`wrapper`直接的函数调用修改为返回一个子函数。这时，我们又得到一个函数，就可以给它继续增加包装了：

```python
def borrow(name,thing):
    print(name,"borrow",thing)

def wrapper(fn):
    def child(*args,**kwds):
        print("before", fn.__name__)
        fn(*args,**kwds)
        print("after",fn.__name__)
    return child

def wrapper2(fn):
    def child(*args, **kwds):
        print("args", args)
        print("kwds", kwds)
        fn(*args, **kwds)
    return child

wrapper2(wrapper(borrow))("skyfire", "pen")
```

此时我们的调用结果入下：

```text
args ('skyfire', 'pen')
kwds {}
before borrow
skyfire borrow pen
after borrow
```

这样，我们就可以使用这种办法不断地给我们的函数加包装了。

但是我们的调用还是不方便，如果可以像加包装前的方式调用就好了，于是我们又做了一点小修改：

```python
def borrow(name,thing):
    print(name,"borrow",thing)

def wrapper(fn):
    def child(*args,**kwds):
        print("before", fn.__name__)
        fn(*args,**kwds)
        print("after",fn.__name__)
    return child

def wrapper2(fn):
    def child(*args, **kwds):
        print("args", args)
        print("kwds", kwds)
        fn(*args, **kwds)
    return child

borrow=wrapper2(wrapper(borrow))

borrow("skyfire", "pen")
```

这次我们使用`borrow`变量覆盖了同名的函数定义，这下完事大吉了～

但是懒惰的程序员并不满足，因为还有一句`borrow=wrapper2(wrapper(borrow))`要写，如果哪天我不想要`wrapper`的包装，只留下`wrapper2`，修改起来还是很麻烦，于是～

**注解出现了～**

来看看我们的终极写法：

```python
def wrapper(fn):
    def child(*args,**kwds):
        print("before", fn.__name__)
        fn(*args,**kwds)
        print("after",fn.__name__)
    return child

def wrapper2(fn):
    def child(*args, **kwds):
        print("args", args)
        print("kwds", kwds)
        fn(*args, **kwds)
    return child

@wrapper2
@wrapper
def borrow(name,thing):
    print(name,"borrow",thing)

borrow("skyfire", "pen")
```

`python`使用`@`来完了我们`borrow=wrapper2(wrapper(borrow))`的代码。
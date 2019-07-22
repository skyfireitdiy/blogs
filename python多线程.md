# python 多线程

## 多线程

```python
import threading


def func(start, end):
    for i in range(start, end):
        print(threading.current_thread().name, i)


def main():
    th = threading.Thread(target=func, args=(0, 10))
    th.start()
    for i in range(10, 20):
        print(threading.current_thread().name, i)
    th.join()


if __name__ == '__main__':
    main()
```

以上程序创建一个子线程，子线程输出0到10的数字，主线程生成10到20的数字。

程序输出：

```text
Thread-1 0
MainThread 10
Thread-1 1
Thread-1 2
Thread-1 3
Thread-1 4
MainThread 11
MainThread 12
Thread-1 5
Thread-1 6
Thread-1 7
Thread-1 8
Thread-1 9
MainThread 13
MainThread 14
MainThread 15
MainThread 16
MainThread 17
MainThread 18
MainThread 19
```

主线程与子线程交替打印。

## 锁

有的时候，多个线程会共享一个资源，这时候可能会发生冲突：

```python
import threading

sum = 0


def func(start, end):
    global sum
    for i in range(start, end):
        sum = sum + i
        sum = sum - i


def main():
    global sum
    th1 = threading.Thread(target=func, args=(0, 1000000))
    th2 = threading.Thread(target=func, args=(0, 1000000))
    th1.start()
    th2.start()
    th1.join()
    th2.join()
    print("sum:", sum)


if __name__ == '__main__':
    main()
```

以上程序，两个线程对同一个变量sum进行操作，都是加某个数，再减掉相同的数。正确的情况下sum应该是0。但以上程序运行几乎每次答案都不一样。

原因是多线程的代码是交替执行的，比如，线程1执行+1然后-1的操作，线程2也执行相同的操作：

线程1：

```Python
_t1 = sum + 1
sum = _t1
_t1 = sum - 1
sum = _t1
```

线程2：

```python
_t2 = sum + 1
sum = _t2
_t2 = sum - 1
sum = _t2
```

如果两个操作以此执行，sum的值会保持不变，但是，如果代码交替执行，就无法保证了：

```python
_t1 = sum + 1   #_t1 = 1
_t2 = sum + 1   #_t2 = 1
sum = _t1       # sum = 1
sum = _t2       # sum = 1
_t1 = sum - 1   #_t1 = 0
sum = _t1       #sum = 0
_t2 = sum - 1   #_t2 = -1
sum = _t2       # sum = -1
```

使用锁就可以轻松解决上面的问题，锁可以保证同一时刻只有一个线程访问被保护的代码段。

```python
import threading

sum = 0
lock = threading.Lock()


def func(start, end):
    global sum
    for i in range(start, end):
        lock.acquire()
        sum = sum + i
        sum = sum - i
        lock.release()


def main():
    global sum
    th1 = threading.Thread(target=func, args=(0, 1000000))
    th2 = threading.Thread(target=func, args=(0, 1000000))
    th1.start()
    th2.start()
    th1.join()
    th2.join()
    print("sum:", sum)


if __name__ == '__main__':
    main()
```

这样，对sum的修改，同一时刻只有一个线程在进行，就可以避免发生竞争了。

## 线程局部存储

当每个线程有自己独立的数据的时候，就需要用上线程记录存储了，线程局部存储就像是一个dict，以线程id为键。

```python
import threading

data = threading.local()


def func(start, end):
    data.sum = 0
    for i in range(start, end):
        data.sum += i
    print(threading.current_thread().name, data.sum)


def main():
    th1 = threading.Thread(target=func, args=(0, 100))
    th2 = threading.Thread(target=func, args=(100, 200))
    th1.start()
    th2.start()
    th1.join()
    th2.join()


if __name__ == '__main__':
    main()
```

在每个线程中，data 的属相都是不同的对象，所以，两个线程互不影响，都能计算出正确答案。

程序输出：

```text
Thread-1 4950
Thread-2 14950
```
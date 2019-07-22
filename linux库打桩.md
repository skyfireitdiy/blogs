# `linux`库打桩

库打桩，就是在`Windows`下常说的`API Hook`，通过替换一些系统的`API`调用从而进行一些监控、统计以及不可告人的目的～～

假设我们有个程序：

```cpp
// filename: main.c

#include <malloc.h>
#include <stdio.h>
#include <string.h>

int main()
{
    char *p = (char*)malloc(256);
    strcpy(p,"hello world\n");
    printf(p);
    free(p);
    return 0;
}
```

编译运行：

```shell
gcc -c main.c
gcc -o main main.o
./main
```

输出结果：

```text
hello world
```

现在我们想监控一下其中的内存分配函数`malloc`和内存释放函数`free`的调用。这里提供三种方法。

## 编译期打桩

我们先写一版我们自己的`malloc`和`free`,需要注意的是，我们的文件名称需要与源文件中引入的`malloc.h`相同，用以欺骗编译器。

```cpp
// filename: malloc.h
#ifndef SF_MALLOC_H_
#define SF_MALLOC_H_

#include <stddef.h>

#ifndef SRC_FLAG
#define malloc our_malloc
#define free our_free
#endif

void *our_malloc(size_t sz);
void our_free(void* p);

#endif
```

接着我们实现`our_malloc`和`our_free`:

```cpp
// filename: malloc.c
#define SRC_FLAG
#include <stdio.h>
#include <malloc.h>

void *our_malloc(size_t sz)
{
    void *p = malloc(sz);
    printf("malloc:%p len:%u\n",p,sz);
    return p;
}

void our_free(void* p)
{
    printf("free:%p\n",p);
    free(p);
}
```

代码中有个`SRC_FLAG`宏需要注意，当定义这个宏的时候，`malloc`和`free`不会被替换，当未定义这个宏的时候`malloc`会被替换为`our_malloc`，`free`会被替换为`our_free`。因为`main.c`中未定义`SRC_FLAG`，所以会发生宏替换，而`malloc.c`中不会发生替换。

接下来我们编译代码（**重要**）：

```shell
gcc -c malloc.c
gcc -I. -o main main.c malloc.o
```

注意这两个编译命令，第一个直接编译`malloc.c`，第二个编译并链接`main.c`和`malloc.o`，注意第二句编译命令，其中有`-I.`参数指定头文件搜索路径为当前目录，此时编译器会优先搜索当前目录，而不是去系统库目录寻找头文件，因此，编译器会使用我们的`malloc.h`，而不是系统的`malloc.h`。

使用了我们的`malloc.h`，`malloc`和`free`就会发生宏替换，从而实现打桩的功能。

第二句编译命令没有使用`-I`参数，是因为我们本意就是使用系统的`malloc.h`，如果指定了`-I.`，那么`malloc.c`中就会包含我们自己写的`malloc.h`，就无法调用`C`库`malloc`和`free`了。

运行一下，查看结果：

```shell
./main
```

输出：

```text
malloc:0x556f8c8a6260 len:256
hello world
free:0x556f8c8a6260
```

内存地址在不同机器上表现不同。

可以看到，我们对源文件`main.c`并没有任何改动，就可以通过宏和编译参数在编译期实现了“打桩”操作。

此方法适合还没有发生任何编译的时期。

接下来我们看一下当只有对象文件`(.o)`的时候如何打桩。

## 链接期打桩

这里我们只有`main.o`文件，而没有`main.c`文件，此时如何打桩呢？

`gcc`有个编译参数：`-Wl`注意，这里的`l`是`L`的小写字母，表示`linker`，而不是`i`的大写或者数字`1`.

使用方法为:

```shell
gcc -Wl,--wrap,func src.o other.o -o bin
```

`-Wl,--wrap,func`表示链接的时候将符号`func`修改为`__wrap_func`，并将`__real_func`修改为`func`。

有了这个操作，我们开始编写代码：

```cpp
// filename: mymalloc.c
#include <stdio.h>
#include <malloc.h>

void* __real_malloc(size_t sz);
void __real_free(void *p);

void* __wrap_malloc(size_t sz)
{
    void *p = __real_malloc(sz);
    printf("malloc:%p len%u\n",p,sz);
    return p;
}

void __wrap_free(void *p)
{
    printf("free:%p\n",p);
    __real_free(p);
}
```

这个源代码中，我们定义了函数`__wrap_malloc`和`__wrap_free`，由上面的解释我们可以知道，在目标文件中的`malloc`和`free`会被替换为这两个函数。这两个函数中调用了`__real_malloc`和`__real_free`，链接时会被替换为`malloc`和`free`，执行真正的内存操作。

接下来我们编译代码：

```shell
gcc -c mymalloc.c
gcc -Wl,--wrap,malloc -Wl,--wrap,free main.o mymalloc.o
```

我们假设`main.o`是由文章开始时的`main.c`使用`gcc -c main.c`编译出来的。

那么链接成功后运行结果为：

```text
malloc:0x55971a754260 len:256
hello world
free:0x55971a754260
```

这就是在没有源码但是有目标文件时的链接期打桩。

但是，如果如果我们连目标文件都没有了，只有最后的可执行文件，是否还可以打桩呢？当然可以！

## 运行期打桩

程序运行后，当遇到未解析的符号时，系统会去搜索`LD_PRELOAD`环境变量包含的库，查找符号，没有找到符号时才会去其他位置查找，我们可以使用这个特性来进行运行期打桩。

编写我们的库代码：

```cpp
// filename: runtime.c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

static int call_printf = 0;

void* malloc(size_t sz)
{
    static int call_time = 0;
    ++call_time;
    void *p;
    void*(*mlc)(size_t) = dlsym(RTLD_NEXT, "malloc");
    char *error;
    if((error=dlerror())!=NULL)
    {
        if(call_printf == 0)
        {
            call_printf = 1;
            printf("Error:%s", error);
            call_printf = 0;
        }
        return NULL;
    }
    p = mlc(sz);
    if(call_printf==0)
    {
        call_printf = 1;
        printf("malloc:%p len:%u\n",p,sz);
        call_printf = 0;
    }
    return p;
}

void free(void *p)
{
    void(*fr)(void*) = dlsym(RTLD_NEXT, "free");
    char * error;
    if((error=dlerror())!=NULL)
    {
        call_printf = 1;
        printf("Error:%s", error);
        call_printf = 0;
        return;
    }
    call_printf = 1;
    printf("free:%p\n",p);
    call_printf = 0;
    fr(p);
}
```

接下来我对代码做一下讲解：

`dlsym`函数用于导出符号，类似于`Windows`下的`GetProcAddress`，其第一个参数是一个共享库的句柄，可以通过`dlopen`函数获取到，本次我们传递了`RTLD_NEXT`参数，这个参数表示从下一个位置搜索符号，什么是下一个位置呢？对于一个符号，`dlsym`会首先从本地搜索，即当前模块（对应`RTLD_DEFAULT`），然后搜索其他（比如`C`库、其他已导入的动态库等），由于我们本模块中已定义了`malloc`和`free`，而我们需要使用`C`库中的`malloc`和`free`，我们自然需要跳过当前模块，即`RTLD_NEXT`。`RTLD_NEXT`在`_GNU_SOURCE`被定义时才生效，其原型为：

```cpp
#define RTLD_NEXT (void*(-1l))
```

另外，注意到我们定义了一个静态变量`call_printf`，这主要是因为`printf`中会调用`malloc`，这时就会发生无限递归，所以我们设置了一个条件来终止递归（事实上添加了这个条件后无法打印`printf`中的`malloc`信息）。

接下来我们来编译动态库：

```shell
gcc runtime.c -shared -fpic -o libruntime.so -ldl
```

其中`-fPIC`表示生成位置无关代码（具体含义自行搜索，只要记住`linux`下的`动态库`基本都需要使用此参数就ok啦），`-ldl`表示链接`dl`库，这个库包含对共享库的操作，`dlsym`就实现在这个库中。

编译后得到`libruntime.so`。

设置环境变量`LD_PRELOAD`（具体的`shell`设置环境变量的方式可能不同，此处以`bash`举例，而且，**因为`LD_PRELOAD`影响较大，千万不要设置系统环境变量，最好只设置当前`shell`或者当前命令范围内的环境变量**）并调用`main`：

```shell
LD_PRELOAD=./libruntime.so ./main
```

得到结果：

```text
malloc:0x563d7aa86260 len:256
hello world
free:0x563d7aa86260
```

这样就完成了运行时的“打桩”。

我们的动态库不止可以在`main`中打桩，还可以在任意程序中打桩，如：

```cpp
LD_PRELOAD=./libruntime.so cat main.c
```

## 总结

* 使用宏和`-I`编译参数对程序进行编译期打桩。
* 使用`-Wl,--wrap,func`对程序进行链接期打桩。
* 使用`LD_PRELOAD`环境变量对程序进行运行期打桩。

于是～～我们就成为了“打桩机”…… ^_^
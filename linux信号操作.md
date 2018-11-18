# linux信号操作

一个信号就是一个小消息，它通知进程系统中发生了一个某种类型的事件。

每种信号类型都对应某种系统事件。底层的硬件异常通常由内核异常处理程序来处理，用户程序对此不可见。信号提供了一种可以通知用户程序的机制，使用户知道发生了这个异常。

接下来我们来介绍下`linux`信号的发送、接收、处理、阻塞、等待等操作。

首先，我们来介绍信号的发送。

## 信号发送

### 使用`/bin/kill`发送信号

`/bin/kill`程序可以向其他进程发送任意指定的信号。使用

```shell
kill -L
```

可以查看系统中的信号：

```text
 1 HUP      2 INT      3 QUIT     4 ILL      5 TRAP
 6 ABRT     6 IOT      7 BUS      8 FPE      9 KILL
10 USR1    11 SEGV    12 USR2    13 PIPE    14 ALRM
15 TERM    16 STKFLT  17 CHLD    17 CLD     18 CONT
19 STOP    20 TSTP    21 TTIN    22 TTOU    23 URG
24 XCPU    25 XFSZ    26 VTALRM  27 PROF    28 WINCH
29 IO      29 POLL    30 PWR     31 SYS     34 RTMIN
64 RTMAX
```

`kill`发送信号的语法是：

```shell
kill -信号 pid
```

或者

```shell
kill -信号 进程名
```

当`pid`的值为负数时，信号会被发送到`-pid`进程组（关于进程组，可自行搜索相关资料）每个进程。

举例，我们启动一个进程`vi`（这个程序基本所有的`linux`发行版都会打包进去）。可以使用`kill -9 vi`向`vi`进程发送`KILL`信号，这时，运行`vi`的窗口会打印出：

```text
fish: Job 2, 'vi' terminated by signal SIGKILL (Forced quit)
```

(不同的终端打印出的信息不同)

`vi`进程收到`KILL`信号后被杀死。

不过，使用进程名发送信号这种方式并不推荐，因为很可能造成“误伤”，可能有多个相同的进程实例，而你可能只是想向其中的一个发送信号，这时可以通过`ps`命令查看要发送的进程的`pid`从而做到“精确投放”。

### 使用键盘发送信号

键盘会向当前只在运行的进程发送一些信号，比如`Ctrl + C`会发送`INT`信号，`Ctrl + Z`会发送`TSTP`信号等。

### 使用`kill`函数

`kill`函数和`kill`命令类似（只是不能接收进程名）。传入参数为进程（组）`pid`和信号值。

```cpp
// filename: mykill.c
#include <signal.h>
#include <stdio.h>
#include <sys/types.h>

int main()
{
    int pid, sig;
    printf("pid:");
    scanf("%d",&pid);
    printf("signal:");
    scanf("%d",&sig);
    kill(pid, sig);
}
```

编译为`mykill`并运行，根据提示输入`pid`和信号值，程序就会给指定的进程（组）发送消息。

### `alarm`给自己发信号

`alarm`函数就像一个闹钟，在指定的时间后，向自己发送一个`ALRM`信号。参数为一个整数，指定延迟的秒数。具体用法可以通过`man alarm`查看。

## 信号接收与处理

`linux`信号通过`signal`函数注册信号处理函数来处理。

下面来举个例子：

```cpp
// filename: myhandler.c
#include <signal.h>
#include <stdio.h>

void sgn60_handler(int sgn_num)
{
    printf("60 signal received\n");
}

void sgn61_62_handler(int sgn_num)
{
    printf("%d signal received\n", sgn_num);
}

void sgn_int_handler(int sgn_int)
{
    printf("sgn int received\n");
}

int main()
{
    // 信号60的响应
    signal(60, sgn60_handler);
    // 信号61的和62的响应，设置为同一个函数
    signal(61, sgn61_62_handler);
    signal(62, sgn61_62_handler);
    // 信号INT的响应，响应Ctrl + C
    signal(SIGINT,sgn_int_handler);
    // 忽略信号TSTP（默认为暂停进程运行）
    signal(SIGTSTP, SIG_IGN);

    getchar();
    return 0;
}
```

例子中我们为信号`60,61,62，INT，TSTP`设置了响应，可以看出同一个响应函数可以对应多个信号，但是注意，**一个信号只能有一个响应**。我们在给`20((SIGTSTP)`设置处理函数时使用了`SIG_IGN`，这个值表示忽略信号，与之类似的还有`SIG_DFL`，表示使用默认处理，不同的信号有不同的默认处理方式。

`signal`函数会返回上一个处理函数的指针，此时我们没有接收使用这个返回值。实际开发中，可能需要保存这个指针用于恢复默认行为。更多`signal`函数的信息，可以使用`man signal`命名查阅文档获得。

编译运行我们的程序，使用`kill`给我们的程序发信号，分别发送`60,61,62,2(SIGINT),20(SIGTSTP)`，我们的程序会有不同的响应。然后再在终端中输入`Ctrl + Z`和`Ctrl + C`（分别对应`2(SIGINT)`和`20(SIGTSTP)`），程序会做出不同的操作。最后我们向程序发送`15(SIGTERM)`，我们的程序会终止，因为我们并没有为`15(SIGTERM)`设置处理函数，而`15(SIGTERM)`的默认处理是终止程序运行。

需要注意的一点是：`SIGKILL`和`SIGSTOP`的默认处理是不能修改的。

**还需要注意的一点是：信号的处理和主程序的逻辑是并行的，所以需要一定的同步机制，此处主程序没有过多的逻辑，所以可以忽略了这一点，实际开发中需要注意**

## 信号阻塞与解除阻塞

* 信号阻塞和忽略的区别

    当信号被阻塞时，信号不会发送至进程，当信号被忽略时，信号仍然会被发送至进程，只是不做任何处理而已。

* 信号阻塞的两种方法

1. 隐式阻塞

    信号在操作系统内核是一系列标记位，每个信号对应一个标记位。当接收到某个信号的时候，内核将该信号的标记位设置为"正在处理"，等到该信号被处理完成后。当一个信号正在被处理的时候，如果再次收到该信号，那么标记位会被设置为“等待处理”，而后续到来的相同信号也只是标记“等待处理”，这就相当于没有被接收。

    举个例子：

    ```cpp
    // filename: block_int.c
    #include <signal.h>
    #include <stdio.h>
    #include <unistd.h>


    void sgn_int_handler(int sgn_int)
    {
        printf("sgn int received\n");
        sleep(3);
    }

    int main()
    {
        // 信号INT的响应，响应Ctrl + C
        signal(SIGINT,sgn_int_handler);
        getchar();
        return 0;
    }
    ```

    以上代码编译运行后，在键盘连续按下多个Ctrl + C（3秒内），最后只会输出两个“sgn int received”，因为当第一个SIGINT触发后，信号开始被处理，后续的多个信号触发的时候，信号只是被简单的设置为“等待处理”，等到第一个信号处理完毕，后续的信号会被认为只有一个待处理信号，所以只会有两个输出。

2. 显式阻塞

    还有一种方式是通过`sigprocmask`函数手动阻塞某种或某几种信号。与之搭配使用的有`sigemptyset`，`sigfillset`,`sigaddset`,`sigdelset`以及一个查询函数`sigismember`。
    
    `sigprocmask`函数传入控制指令和一个信号集合，返回原信号集合（通过参数返回）。控制指令包括`SIG_BLOCK`（添加阻塞信号集合），`SIG_UNBLOCK`（删除阻塞信号集合），`SIG_SETMASK`（设置阻塞信号集合）。

    其他的配套函数对信号集合进行操作。

    函数的具体说明可以参考文档。

    这里我们举个例子阻塞`SIGINT`和`SIGTSTP`信号。

    ```cpp
    // filename: block_int_tstp.c
    #include <signal.h>
    #include <stdio.h>
    #include <unistd.h>


    void sgn_int_handler(int sgn_int)
    {
        printf("sgn int received\n");
    }

    void sgn_tstp_handler(int sgn_tstp)
    {
        printf("sgn tstp received\n");
    }

    int main()
    {
        sigset_t mask, premask;
        // 信号INT的响应，响应Ctrl + C
        signal(SIGINT,sgn_int_handler);
        // 信号TSTP的响应，响应Ctrl + Z
        signal(SIGINT,sgn_tstp_handler);
        
        sigemptyset(&mask);
        sigaddset(&mask, SIGINT);
        sigaddset(&mask, SIGTSTP);
        
        sigprocmask(SIG_BLOCK, &mask, &premask);

        getchar();
        return 0;
    }
    ```

    程序编译运行后，在终端按下Ctrl+C和Ctrl+Z后，应用程序不反应，说明我们阻塞了SIGINT和SIGTSTP信号。

## 等待信号

### 使用`pause`等待信号

`pause`会挂起当前进程，直到一个信号到来。下面演示一个例子。

```cpp
// filename: pause_demo.c
#include <unistd.h>
#include <stdio.h>
#include <signal.h>

void int_handler(int sig_num)
{
}

int main()
{
    signal(SIGINT,int_handler);
    pause();
    printf("hello world\n");
    return 0;
}
```

编译运行程序后，程序不会退出，按下`Ctrl+C`发送`SIGINT`，然后程序打印`hello world`后退出。

### 使用`sigsuspend`函数等待信号

与`pause`不同的是，`pause`会等待任意信号，而`sigsuspend`等待指定类型的信号。

```cpp
// filename: sigsuspend_demo.c
#include <unistd.h>
#include <stdio.h>
#include <signal.h>

void my_handler(int sig_num)
{
    printf("%d sig\n",sig_num);
}

int main()
{
    sigset_t mask;
    
    sigfillset(&mask);
    sigdelset(&mask, SIGINT);
    sigdelset(&mask, SIGTSTP);

    signal(SIGINT, my_handler);
    signal(SIGTSTP,my_handler);
    signal(60,my_handler);
    
    sigsuspend(&mask);

    printf("hello world\n");
    return 0;
}
```

编译运行，使用`kill`分别发送`60`、`SIGINT`，发送`60`的时候程序没反应（因为已经被挂起），发送`SIGINT`的时候，程序恢复执行，并且之前的`60`信号也被执行（其实早已经执行，但是当前进程被挂起，所以看不到输出）。

`sigsuspend`函数相当于将`mask`中标记的信号全部阻塞，然后调用`pause`，然后调用`pause`，等待到信号后，再回复现场。


本文章简单地讨论了`linux`下信号的使用，但是还有很多没有提到的细节，而且例子也大多不规范（只为了说明问题），更多细节请参考官方的文档或者更权威的书籍。
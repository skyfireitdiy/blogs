# C++11 extern template

PS:不知道对应的中文术语叫啥（模板声明？）

模板是泛型编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

模板是创建泛型类或函数的蓝图或公式。库容器，比如迭代器和算法，都是泛型编程的例子，它们都使用了模板的概念。

使用模板可以少写很多冗余代码，但是同样会带来一些开销：

* 中间代码体积变大
* 编译时间增长

我们来看一个例子（这个例子涉及到5个文件，可能会比较繁琐）

h.h
```cpp
template <typename T>
void my_swap(T &a, T &b)
{
    if (a == b)
    {
        return;
    }
    auto t = a;
    a = b;
    b = t;
}
```

h.h中定义了一个模板函数`my_swap`用于交换两个值。

a.cpp

```cpp
#include "h.h"

void swap_one(int &a, int &b)
{
    my_swap(a, b);
}
```

b.cpp
```cpp
#include "h.h"

void swap_two(int &a, int &b)
{
    my_swap(a, b);
}
```

a.cpp和b.cpp中分别有两个函数swap_one和swap_two,作用是一致的，都是交换两个整数，并且在内部都调用了`my_swap`模板函数。

main.cpp

```cpp
#include <iostream>

using namespace std;

void swap_one(int &, int &);
void swap_two(int &, int &);

int main()
{
    int a = 5, b = 10;
    swap_one(a, b);
    swap_two(a, b);
}
```

main.cpp对a.cpp和b.cpp的两个函数进行调用。

对于这样一个工程，使用`g  `去编译生成汇编代码，看看发生了什么：

```bash
g++ a.cpp -o a.asm -O0 -S -c
g++ b.cpp -o b.asm -O0 -S -c
```

生成的a.asm和b.asm如下：

a.asm

```asm
	.file	"a.cpp"
	.text
	.globl	_Z8swap_oneRiS_
	.type 	_Z8swap_oneRiS_, @function
_Z8swap_oneRiS_:
.LFB1:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movq	%rdi, -8(%rbp)
	movq	%rsi, -16(%rbp)
	movq	-16(%rbp), %rdx
	movq	-8(%rbp), %rax
	movq	%rdx, %rsi
	movq	%rax, %rdi
	call	_Z7my_swapIiEvRT_S1_
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size   	_Z8swap_oneRiS_, .-_Z8swap_oneRiS_
	.section	.text._Z7my_swapIiEvRT_S1_,"axG",@progbits,_Z7my_swapIiEvRT_S1_,comdat
	.weak   	_Z7my_swapIiEvRT_S1_
	.type   	_Z7my_swapIiEvRT_S1_, @function
_Z7my_swapIiEvRT_S1_:
.LFB2:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movq	%rdi, -24(%rbp)
	movq	%rsi, -32(%rbp)
	movq	-24(%rbp), %rax
	movl	(%rax), %edx
	movq	-32(%rbp), %rax
	movl	(%rax), %eax
	cmpl	%eax, %edx
	je  	.L5
	movq	-24(%rbp), %rax
	movl	(%rax), %eax
	movl	%eax, -4(%rbp)
	movq	-32(%rbp), %rax
	movl	(%rax), %edx
	movq	-24(%rbp), %rax
	movl	%edx, (%rax)
	movq	-32(%rbp), %rax
	movl	-4(%rbp), %edx
	movl	%edx, (%rax)
	jmp 	.L2
.L5:
	nop
.L2:
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE2:
	.size   	_Z7my_swapIiEvRT_S1_, .-_Z7my_swapIiEvRT_S1_
	.ident  	"GCC: (Debian 8.3.0-19) 8.3.0"
	.section	.note.GNU-stack,"",@progbits
```

b.asm如下

```asm
	.file	"b.cpp"
	.text
	.globl	_Z8swap_twoRiS_
	.type 	_Z8swap_twoRiS_, @function
_Z8swap_twoRiS_:
.LFB1:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movq	%rdi, -8(%rbp)
	movq	%rsi, -16(%rbp)
	movq	-16(%rbp), %rdx
	movq	-8(%rbp), %rax
	movq	%rdx, %rsi
	movq	%rax, %rdi
	call	_Z7my_swapIiEvRT_S1_
	nop
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size   	_Z8swap_twoRiS_, .-_Z8swap_twoRiS_
	.section	.text._Z7my_swapIiEvRT_S1_,"axG",@progbits,_Z7my_swapIiEvRT_S1_,comdat
	.weak   	_Z7my_swapIiEvRT_S1_
	.type   	_Z7my_swapIiEvRT_S1_, @function
_Z7my_swapIiEvRT_S1_:
.LFB2:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movq	%rdi, -24(%rbp)
	movq	%rsi, -32(%rbp)
	movq	-24(%rbp), %rax
	movl	(%rax), %edx
	movq	-32(%rbp), %rax
	movl	(%rax), %eax
	cmpl	%eax, %edx
	je  	.L5
	movq	-24(%rbp), %rax
	movl	(%rax), %eax
	movl	%eax, -4(%rbp)
	movq	-32(%rbp), %rax
	movl	(%rax), %edx
	movq	-24(%rbp), %rax
	movl	%edx, (%rax)
	movq	-32(%rbp), %rax
	movl	-4(%rbp), %edx
	movl	%edx, (%rax)
	jmp 	.L2
.L5:
	nop
.L2:
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE2:
	.size   	_Z7my_swapIiEvRT_S1_, .-_Z7my_swapIiEvRT_S1_
	.ident  	"GCC: (Debian 8.3.0-19) 8.3.0"
	.section	.note.GNU-stack,"",@progbits
```

从汇编代码中，可以看出，在a.cpp和b.cpp中，都有`_Z7my_swapIiEvRT_S1_`这个符号，这个符号其实就是`my_swap` 被实例化后的符号，也就是说，两个编译单元中都对`my_swap`函数进行了实例化，这不仅增加了中间目标文件的体积，更增加了编译时间。

而每个编译单元对相同模板函数的相同类型实例化，而实例化后的符号为弱符号（不清楚弱符号的童鞋自行[百度](https://www.baidu.com)），最后链接也只会留下一份。所以实际上只需要做一遍实例化就可以了。所以`C++11`增加了`extern template`，带有`extern template`声明的模块在编译的时候回跳过指定模板的实例化，链接时会去其他模块寻找相关的实例化代码。

将a.cpp修改为以下代码：

```cpp
#include "h.h"

extern template void my_swap<int>(int &, int &);

void swap_one(int &a, int &b)
{
    my_swap(a, b);
}
```

重新编译a.cpp为汇编代码a.asm：

```asm
	.file	"a.cpp"
	.text
	.globl	_Z7funcIntv
	.type 	_Z7funcIntv, @function
_Z7funcIntv:
.LFB1:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	call	_Z4funcIiEPvv@PLT
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size   	_Z7funcIntv, .-_Z7funcIntv
	.ident  	"GCC: (Debian 8.3.0-19) 8.3.0"
	.section	.note.GNU-stack,"",@progbits
```

可以看到a.asm中不再有 `my_swap` 的实例化代码。链接的时候会去其他模块寻找相应实现。

测试一下编译：

```
g++ a.cpp -c
g++ b.cpp -c
g++ main.cpp -c
g++ main.o a.o b.o -o main
```

编译顺利通过。

从上面可以看出，`extern template`减少了模板类型的实例化代码编译次数，也就意味着节省了时间与空间，鱼与熊掌兼得。

使用`extern template`需要注意的地方：

1. 必需确保`extern`的模板在其他编译单元中有实例化。例如，不能同时在a.cpp和b.cpp同时使用`extern template`，否则会导致模板从未被实例化，会引起`undefined reference`的问题。
2. 必需确保`extern`的类型与外部的类型一致，否则不起效果。比如a.cpp中的`extern template`修改为：
```cpp
extern template void my_swap<double>(double &, double &);
```
这样在编译到`my_swap(a,b)`的时候，同样会再实例化`my_swap<int>`的版本。


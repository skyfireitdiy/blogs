# Modern-Cpp

**skyfire**

*2018/01/30*

Talk is cheap, show me the code !

笔记中使用的编译环境为 gcc 7.2/gcc 7.1([Windows MinGW64 下载](http://www.mingw-w64.org/doku.php), cmake 3.7([CMake 下载](https://cmake.org/))

此笔记仅作简要介绍和粗浅示例，需要了解具体细节，可前往[cppreference.com](http://zh.cppreference.com)

## 1.alignas

`alignas`用于指定内存对齐。

* 使用示例1：

```cpp
#include <iostream>

using namespace std;

//指定该结构体以4字节对齐
struct alignas(4) Stu1
{
    char a;
};

struct Stu2
{
    char a;
};

int main()
{
    cout<<sizeof(Stu1)<<endl;
    cout<<sizeof(Stu2)<<endl;
}
```

程序输出结果：

```text
4
1
```

针对结构体(类)直接指定`alignas`，使其内部成员按照指定的对齐方式进行对齐。

* 使用示例2：

```cpp
#include <iostream>

using namespace std;

//指定该结构体以4字节对齐
struct alignas(4) Stu1
{
    char a;
};

int main()
{
    cout<<sizeof(Stu1)<<endl;
    alignas(4) char arr[1];
    cout<<sizeof(arr)<<endl;
}
```

程序输出：

```text
4
1
```

对变量(数组)使用`alignas`指定对齐方式，作用为使其分配内存的地址以该字节对齐，但是其占用空间b并非按照指定的对齐方式对齐。

* 使用示例3

```cpp
#include <iostream>

using namespace std;

//指定该结构体以4字节对齐
struct alignas(4) Stu1
{
    char a;
};

// 按照Stu1相同的方式对齐
struct alignas(Stu1) Stu2
{
    char a;
};

int main()
{
    cout<<sizeof(Stu1)<<endl;
    cout<<sizeof(Stu2)<<endl;
}
```

程序输出

```text
4
4
```

使用`alignas(类型)`，会以指定类型的对齐方式指定对齐方式。相当于：

```cpp
alignas(alignof(类型))
```

## 2.alignof

`alignof`用于获取对象（类）的对齐方式。

* 使用示例

```cpp
#include <iostream>
 
struct Foo {
    int   i;
    float f;
    char  c;
};
 
struct Empty {};
 
struct alignas(64) Empty64 {};
 
int main()
{
    std::cout << "Alignment of"  "\n"
        "- char             : " << alignof(char)    << "\n"
        "- pointer          : " << alignof(int*)    << "\n"
        "- class Foo        : " << alignof(Foo)     << "\n"
        "- empty class      : " << alignof(Empty)   << "\n"
        "- alignas(64) Empty: " << alignof(Empty64) << "\n";
}
```

程序输出：

```text
Alignment of
- char             : 1
- pointer          : 4
- class Foo        : 4
- empty class      : 1
- alignas(64) Empty: 64
```

## 3.auto

`auto`的作用为类型自动推导

* 使用示例1

```cpp
int main()
{
    auto a = 0;
    auto b = 1.5;
    auto c = "hello", d = "world";
}
```

上例中，`a`被推导为`int`，`b`为`double`，`c`和`d`为`const char*`类型。

**注意**

*当使用`auto`同时声明多个变量时，多个变量推导的类型必须完全一致。*

* 使用示例2

```cpp
#include <iostream>

using namespace std;

auto add(int a,int b) -> int
{
    return a+b;
}

int main()
{
    cout<<add(5,7)<<endl;
}
```

程序输出：

```text
12
```

`auto`可用于返回类型后置的函数，此时`auto`不再起类型推导的作用，而只是语法的一部分。

* 使用示例3

```cpp
#include <iostream>

using namespace std;

auto calc(int a,int b,bool op_add)
{
    if(op_add)
    {
        return a+b;
    }
    else
    {
        return a*b;
    }
}

int main()
{
    cout<<calc(5,9,true)<<endl;
    cout<<calc(5,9,false)<<endl;
}
```

程序输出：

```text
14
45
```

当`auto`被用于不使用尾随返回类型语法的函数声明中时，`auto`的类型通过`return`语句推导，当有多个`return`语句时，每个`return`推导出的类型必须相同。

* 使用示例4

```cpp
#include <iostream>

using namespace std;

int main()
{
    int a = 5;
    int &b = a;
    decltype(auto) c = b;
    c = 20;
    cout<<a<<endl;
}
```

程序输出：

```text
20
```

使用`decltype(auto)`声明变量（`decltype`将在后续小节中讲解），`auto`位置将会被变量的赋值表达式替代，实际的类型推导由`decltype`完成。此例子中会保留引用属性。

* 使用示例5

```cpp
#include <iostream>

using namespace std;

decltype(auto) myref(int &r)
{
    return r;
}

int main()
{
    int a = 5;
    myref(a) = 10;
    cout<<a<<endl;
}
```

程序输出

```text
10
```

若声明函数返回类型为`decltype(auto)` ，则关键词`auto`被替换成其返回语句的运算数，而其实际返回类型以`decltype`规则推导。

* 使用示例6

```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
    auto lbd = [](auto a, auto b) -> decltype(a)
    {
        return a+b;
    };
    cout<<lbd(5,6)<<endl;
    cout<<lbd(string("hello"),string(" world"))<<endl;
}
```

程序输出：

```text
11
hello world
```

当`auto`用在`lambda`表达式参数列表中时，`lambda`表达式的参数会在调用时根据实参进行推导（泛型`lambda`）。

* 使用示例7

```cpp
#include <iostream>

using namespace std;

template<auto n>
decltype(n) func(decltype(n) a,decltype(n) b)
{
    return a+b;
}

int main()
{
    cout<<func<5>(6,7)<<endl;
}
```

程序输出

```text
13
```

若模板形参声明为 auto ，则其类型由对应使用参数推导出。

* 使用示例8

```cpp
#include <iostream>
#include <string>

using namespace std;

struct Stu
{
    int a;
    double b;
    string c;
};

int main()
{
    Stu st1 = {5,3.5,"hello world"};
    auto [a,b,c] = st1;
    cout<<a<<endl;
    cout<<b<<endl;
    cout<<c<<endl;
}
```

程序输出：

```text
5
3.5
hello world
```

`auto`作为结构化绑定的类型声明。（结构化绑定在后续小节中介绍）

## 4.decltype

`decltype`用于类型推导。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    int a = 5;
    decltype(a) b = a;
    b = 10;
    cout << "a:" << a <<endl;
    cout << "b:" << b <<endl;
    decltype((a)) c = a;
    c = 20;
    cout << "a:" << a <<endl;
    cout << "c:" << c <<endl;
}
```

程序输出：

```text
a:5
b:10
a:20
c:20
```

`decltype(a)`会推导出`a`的类型，`decltype((a))`则会推导出`a`的引用类型。

* 使用示例2

```cpp
#include <iostream>

using namespace std;

template<typename A, typename B>
auto add(A a, B b) -> decltype(a+b)
{
    return a+b;
}

int main()
{
    int a = 5;
    double b = 6.5;
    cout<<add(a,b)<<endl;
}
```

程序输出

```text
11.5
```

`decltype`用来推导表达式类型的时候，不会计算表达式，只会推导表达式的最终返回类型。

## 5.constexpr

`constexpr`提供编译期常量。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

constexpr int const_func(int param)
{
    return param + 1;
}

template<int N>
void func()
{
    cout<<N<<endl;
}

int main()
{
    func<const_func(5)>();
}
```

程序输出：

```text
6
```

上例中创建了一个`constexpr`函数，此函数返回一个编译期常量，该常量被用来作为模板参数。

`constexpr`函数的参数必须为`constexpr`类型或者字面值。（即编译期可知）

* 使用示例2

```cpp
#include <iostream>

using namespace std;

class const_class
{
private:
    int sum;

    constexpr int calc_sum(int n) const
    {
        return n == 1 ? 1 : n + calc_sum(n - 1);
    }

public:
    constexpr const_class(int n):sum(calc_sum(n))
    {
    }

    constexpr get_sum() const
    {
        return sum;
    }
};


int main()
{
    constexpr const_class c(10);
    cout<<c.get_sum()<<endl;
}
```

程序输出

```text
55
````

上例中定义了`constexpr`构造函数，同时定义了`constexpr`成员函数。

关于`constexpr`更多限制与规定，请参考[cppreference.com](http://en.cppreference.com/w/cpp/language/constexpr)


## 6.默认化和被删除的函数（default_and_delete）

`default`使类生成默认构造函数，`delete`删除指定的构造函数。

* 使用示例1

```cpp
class A
{
public:
    A(int){} // 定义了带参数的构造函数，则默认构造函数不会生成
};

class B
{
public:
    B(int){}
    B() = default;
};

class C
{

};

class D
{
public:
    D() = delete;
};

int main()
{
    // A a;     // error,没有A::A()
    B b;        // ok, 由default生成默认构造函数
    C c;        // ok, 生成默认构造函数
    // D d;     // error, delete可删除指定的构造函数
}
```

例子中给出了`default`和`delete`的使用方法和作用。

## 7.委托构造函数与继承构造函数（constructors）

委托构造函数允许构造函数中调用其他构造函数，继承构造函数允许继承父类的构造函数。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

class A
{
public:
    A(){
        cout<<"A()"<<endl;
    }

    A(int):A()      //委托构造，调用A()
    {
        cout<<"A(int)"<<endl;
    }
};

class B: public A
{
    using A::A;     //继承构造函数，将类A的构造函数继承过来
};

int main()
{
    A a(5);
    B b(5);
}
```

程序输出：

```text
A()
A(int)
A()
A(int)
```

例子中，`A::A(int)`调用`A()`来进行初始化（委托构造），而类`B`直接继承类`A`的构造函数。

## 8.显式转换运算符（explicit_conversion_operator）

显式转换运算符去除运算符的隐式转换。

* 使用示例

```cpp
#include <iostream>

using namespace std;

class A{};

class B{};

class C{
public:
    operator A()
    {
        return A();
    }

    explicit operator B()       //指定显式转换，去除隐式转换
    {
        return B();
    }
};

int main()
{
    C c;
    A a = c;
    // B b = c;                 // error, 不存在从C到B的隐式转换
    B bb = static_cast<B>(c);   // ok, 可以显示转换
}
```

## 9.扩展的友元（friend）

`C++11`中对`friend`关键字进行了扩展，使其可省略`class`关键字，并且可以使用别名。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

class B;
class C;
typedef C CC;

class A
{
private:
    int a = 5;
public:
    void print_a()
    {
        cout << a << endl;
    }
    friend B;           // 在C++11前，必须为friend class B;
    friend CC;          // C++11前，不能用别名
};

class B
{
public:
    void increment(A &t)
    {
        ++t.a;          // 访问类A的私有成员
    }
};

class C
{
public:
    void increment(A &t)
    {
        ++t.a;          // 访问类A的私有成员
    }
};

int main()
{
    A a;
    a.print_a();
    B b;
    C c;
    b.increment(a);
    a.print_a();
    c.increment(a);
    a.print_a();
}
```

程序输出：

```text
5
6
7
```

## 10.外部模板声明（extern_template）

外部模板声明是为了告知编译器，此模板已特化，不需要再次特化，减少编译时间。

* 使用示例1

`A.h`

```cpp
#pragma once

template<typename T>
class A{};
```

`B.h`

```cpp
#pragma once
#include <A.h>

extern template class A<int>;       // 告知编译器，A<int>已经在其他编译单元进行特化

template<typename T>
class B : public A<T>
{

};
```

`A.cpp`

```cpp
#include <A.h>

A<int> a;           //这里对A<int>进行特化
```

`test_extern_template1.cpp`

```cpp
#include <B.h>

int main()
{
    B<int> b;
}
```

例子中编译单元`A.cpp`会对模板`A<int>`进行特化，而`B.h`中不会再次特化。

## 11.枚举前向声明

`C++11`支持枚举前向声明。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

enum E : short;     // 枚举的前向声明（需要指定基类型）

void func(E a)
{
    cout<<a<<endl;
}

enum E : short
{
    E1,
    E2
};

int main()
{
    func(E1);
}
```

## 12.列表初始化（list_initialization）

`C++11`的列表初始化为对象、参数等的初始化提供便利。

* 使用示例1

```cpp
#include <iostream>
#include <map>
#include <string>
#include <vector>

using namespace std;

class A
{
private:
    vector<int> data1;
    vector<int> data2{1,2,3,4,5,6};         //类成员列表初始化（类内初始化）
    vector<int> data3 = {1,2,3,4,5,6};      //类成员列表初始化（类内初始化）(使用等号)
public:
    A(vector<int>,int) : data1{1,2,3,4,5,6}{}              //类成员列表初始化（构造函数初始化）
    bool operator [] (vector<int>){}
    A& operator = (vector<int>)
    {
        return *this;
    }
};

vector<int> func(vector<int> t)
{
    return {1,2,3,4,5,6};                  // 使用列表初始化构造返回值
}

int main()
{
    vector<int> a {1,2,3,4,5,6,7};          //初始化具名变量
    auto b = vector<int>{1,2,3,4,5,6,7};    //初始化临时变量
    auto c = new vector<int>{1,2,3,4,5,6,7};//动态分配初始化
    delete c;

    vector<int> d = {1,2,3,4,5,6,7};        //使用赋值运算符进行列表初始化
    func({1,2,3,4,5,6,7});                  //参数传递，使用列表初始化构造参数

    A e {{1,2,3,4,5,6},5};                  //生成构造函数的参数
    e = {2,3,4,5,6,7};                      //生成operator = 的参数
    e[{1,2,3,4,5,6}];                       //列表初始化构造下标

}
```

## 13.lambda

`lambda`表达式提供了匿名函数的实现，可以在使用很多STL算法时不必要专门定义一个具名函数。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    int a = 5;
    int b = 10;
    auto lfun = [&,a](int c, auto d) -> decltype(a+b+c+d) {
        //a = 1;                // a的值为只读，不能修改
        b = 2;
        return a+b+c+d;
    };

    cout << lfun(15,3.5)<<endl;
    cout << "a:"<<a<<endl;
    cout << "b:"<<b<<endl;
}
```

程序输出：

```text
25.5
a:5
b:2
```

`lambda`表达式一般由以下部分构成：(以下内容引用自[cppreference.com](http://en.cppreference.com/w/cpp/language/lambda))

```cpp
[ captures ] <tparams>(可选)(C++20) ( params ) specifiers(可选) exception attr -> ret requires(可选)(C++20) 
{
    body
} 
```

`captures` 	- 	零或更多捕获的逗号分隔列表，可选地以 `capture-default` 起始。

捕获列表能按如下方式传递（详细描述见下方）：

    [a,&b] 其中 a 以复制捕获而 b 以引用捕获。
    [this] 以引用捕获当前对象（ *this ）
    [&] 以引用捕获所有用于 lambda 体内的自动变量，并以引用捕获当前对象，若存在
    [=] 以复制捕获所有用于 lambda 体内的自动变量，并以引用捕获当前对象，若存在
    [] 不捕获 

若变量满足下列条件，则 `lambda` 表达式能使用而不捕获它

    为非局部变量，或拥有静态或线程局域存储期（该情况下不能捕获该变量），或
    为以常量表达式初始化的引用。 

若变量满足下列条件，则 lambda 表达式能读取其值而不捕获它

    拥有 const 而非 volatile 的整数或枚举类型，并已用常量表达式初始化，或
    为 constexpr 且为可平凡复制构造。 

不能捕获结构化绑定。

`<tparams>(C++20)` 	- 	模板形参列表（于角括号中），用于提供名称给泛型 `lambda` 的模板形参。类似在模板声明中，模板形参列表可以后附可选的 `requires` 子句，它指定模板实参上的制约。

`params` 	- 	参数列表，如在具名函数中，除了不允许默认参数 (C++14 前)。若以 `auto` 为参数类型，则该 `lambda` 为泛型 `lambda` 。 (C++14 起)
`specifiers` 	- 	可选的指定符序列。允许下列指定符：

    mutable ：允许 body 修改以复制捕获的参数，及调用其非 const 成员函数 

    constexpr ：显式指定函数调用运算符为 constexpr 函数。此指定符不存在时，若函数调用运算符恰好满足所有 constexpr 函数要求，则它也会是 constexpr 


`exception` 	- 	为闭包类型的 `operator()` 提供异常规定或 `noexcept` 子句

`attr` 	- 	为闭包类型的 `operator()` 提供属性指定

`ret` 	- 	返回类型。若缺失，则由函数的 `return` 语句所隐含（或若函数不返回任何值则为 `void` ）

`body` 	- 	函数体 


 * C++20的泛型lambda

 ```cpp
// 泛型 lambda ， operator() 是有二个形参的模板
auto glambda = []<class T>(T a, auto&& b) { return a < b; };

// 泛型 lambda ， operator() 是有一个形参包的模板
auto f = []<typename ...Ts>(Ts&& ...ts) {
    return foo(std::forward<Ts>(ts)...);
};
 ```

## 14.局部及无名类型作为模板形参（Local and Unnamed Types as Template Arguments）

`C++11`允许无名类型作为模板的形参，无名类型的作用域为本编译单元，不同编译单元的无名类型不冲突。

* 使用示例1

`test_local_and_unamed_types_as_template_arguments1.cpp`

```cpp
template<typename T>
void tfunc(T t)
{
}

enum ttt{E1};          //无名枚举
struct {} S1;       //无名结构体

int main()
{
    tfunc(E1);
    tfunc(S1);
}
```

`test_local_and_unamed_types_as_template_arguments2.cpp`

```cpp
enum {E1};          //与另一文件定义的无名变量是两个类型，不会发生链接错误
struct {} S1;
```

两个文件可正常编译链接。

## 15.long long

`C++11`引入`long long`长整型，`64bit`长度。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    long long a = 5LL;
    cout << sizeof(a) << endl;
}
```

程序输出：

```text
8
```

## 16.内联命名空间（inline namespace）

`C++11`支持内联命名空间，使用起来如同外部命名空间。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

namespace A
{
    void func1()
    {
        cout<<"func1"<<endl;
    }
    inline namespace
    {
        void func2()
        {
            cout<<"func2"<<endl;
        }
    }
}

int main()
{
    A::func1();
    A::func2();     //使用了内联命名空间，如同在A中一样
}
```

程序输出

```text
func1
func2
```

## 17.新字符类型（new char）

`C++11`新增了`char16_t`和`char32_t`类型，用字面值前面带上`u`和`U`来表示。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout<<sizeof(char)<<endl;
    cout<<sizeof(wchar_t)<<endl;
    cout<<sizeof(char16_t)<<endl;
    cout<<sizeof(char32_t)<<endl;
    char a[16] = "hello world";
    wchar_t b[16] = L"hello world";
    char16_t c[16] = u"hello world"; 
    char32_t d[16] = U"hello world";
}
```

程序输出

```text
1
2
2
4
```

## 18.函数尾置返回类型（new return）

`C++11`支持函数返回类型后置，这将使返回类型依赖于参数类型的函数更便于编码。

* 使用示例1

```cpp
#include <iostream>
#include <string>

using namespace std;

template<typename T>
auto tfunc(int a, double b, T Func) -> decltype(Func(a,b))  //返回类型只有实例化时才能知晓
{
    return Func(a,b);
}

auto func1(int a, double b) -> decltype(a+b)     //需要根据参数类型推导
{
    return a+b;
}

auto func2(int a,double b) -> string            //尾置返回string
{
    string ret;
    for(auto i = 0;i < a; ++i)
    {
        ret += to_string(b);
    }
    return ret;
}

auto main() -> int
{
    cout<<tfunc(2,3.5,func1)<<endl;
    cout<<tfunc(2,3.5,func2)<<endl;
}
```

程序输出

```text
5.5
3.5000003.500000
```

例子中模板函数的返回类型依赖于模板参数类型组合（调用）的结果，此时使用尾置返回类型会更加方便。

## 19.nullptr

`C++11`不推荐再继续使用`NULL`宏或`0`来表示空指针，而是应该使用`nullptr`表示空指针，这是一种全新的类型（`nullptr_t`）

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    int *p = nullptr;       // 使用nullptr表示空指针
    int a = 5;
    p = &a;
    *p = 10;
    cout << a << endl;
}
```

程序输出

```text
10
```

## 20.原始字符串与Unicode 字符串字面量（unicode string literals）

`C++11`增加了原始字符串，同时增加了对应的`Unicode`版本.

原始字符串定义为`R"xxx(raw string)xxx"`其中`xxx`为任意字符串，且左右相等，如：`R"line(my string)line"`等价于`"my string"`

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    char raw_str[] = R"*(C:\Windows\System32\next)*";     //不会转义
    cout << raw_str << endl;

    char16_t uraw_str[] = uR"line(
        第一行
        第二行
        )line";                                            //原始字符串可以跨行 
}
```

程序输出

```text
C:\Windows\System32\next
```

## 21.用户自定义字面量（User defined Literals）

`C++11`可以让用户自定义字面值。

* 使用示例1

```cpp
#include <iostream>
#include <string>

using namespace std;

long long operator "" _k(unsigned long long t)      //数字乘以1000
{
    return 1000*t;
}

string operator "" _add_prefix(const char * p, size_t)     //处理字符串
{
    return string("Tom say:") + p;
}

int main()
{
    cout << 10_k <<endl; 
    cout << "hello world"_add_prefix << endl;
}
```

程序输出：

```text
10000
Tom say:hello world
```

`operator ""`的参数有以下几种类型：

```cpp
char const*                 // 12_k  ->  operator "" _k("12")
unsigned long long int      // 12_k  ->  operator "" _k(12)
long double                 // 1.2_K ->  operator "" _k(1.2)
char const*, std::size_t    // "12"_K ->  operator "" _k("12", 2)
wchar_t const*, std::size_t // L"12"_k -> operator "" _k(L"12", 2)
char16_t const*, std::size_t // u"12"_k -> operator "" _k(u"12", 2)
char32_t const*, std::size_t // U"12"_k -> operator "" _k(U"12", 2)
```

## 22.右角括号（Right Angle Brackets）

`C++11`放宽了模板嵌套时右角括号的限制，使代码看起来更美观。

* 使用示例1

```cpp
#include <vector>

using namespace std;

int main()
{
    vector<vector<int>> arr;        // C++11之前只能这样写：vector<vector<int> > arr;
}
```

## 23.右值引用（Rvalue Reference）

`C++11`增加了对右值的引用。右值，可理解为“没有名字的值”，如函数返回的临时对象、字面值等。而右值引用是为了延长这些对象的生存期。

* 使用示例1:

```cpp
#include <iostream>

using namespace std;

int add(int a, int b)
{
    return a + b;
}

void func(int&)
{
    cout<<"lvalue"<<endl;
}

void func(int&&)
{
    cout<<"rvalue"<<endl;
}

int main()
{
    int a = 2., b = 3;
    func(0);
    func(a);
    func(add(a,b));
}
```

程序输出：

```text
rvalue
lvalue
rvalue
```

从输出可以看到，字面值、函数的返回值会匹配`int&&`(右值引用)，而命名对象，则会匹配`int&`(左值引用)。右值引用使即将被销毁的存储区再次被利用起来，提高了对象利用率。

## 24.静态断言（Static Assert）

`C++11`增加静态断言，使程序在编译期可以检测一些条件。

* 使用示例

```cpp
int main()
{
    static_assert(sizeof(void*) == 4, "please use x86 compiler");
}
```

此程序如果使用x64编译器编译，会出现错误：

```text
error: static assertion failed: please use x86 compiler
```

## 25.枚举类（Enum Class）

`C++11`增加枚举类概念，使枚举的使用更加可控。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

enum class E1   //定义枚举类
{
    E1a,
    E1b
};

enum class E2 : char    // 定义枚举类，以char为其基类型
{
    E2a = 'a',
    E2b = 'b'
};

int main()
{
    E1 e = E1::E1a;
    // int n = e;           //错误，不能进行隐式转换
    cout << static_cast<int>(e) << endl;
    cout << static_cast<char>(E2::E2a) << endl;
}
```

程序输出

```text
0
a
```

## 26.模板别名（Templates Aliases）

`C++11`可以为模板定义别名，省去了编写太长模板参数的麻烦。

* 使用示例1

```cpp
#include <vector>

using namespace std;

int main()
{
    using vec_int_int = vector<vector<int>>;        // 后面就可以用vec_int_int替代vector<vector<int>>使用了
    vec_int_int a(5);
    a[0].push_back(654);
}
```

## 27.线程局部存储（Thread-Local Storage）

`C++11`引入线程局部存储，为多线程数据隔离提供支持。

* 使用示例1

```cpp
#include <thread>
#include <iostream>
#include <vector>
#include <mutex>

using namespace std;

thread_local long long sum = 0;           // 线程局部存储变量，在每个线程中都是独立的
mutex mu;                           // 互斥量，用于对cout输出进行控制

void thread_func()                  //线程执行函数，计算1到1000000的和
{
    for(int i = 1;i < 1000000; ++i)
    {
        sum += i;                   // 此处的sum在每个线程中都有一个实例，不会相互影响
    }
    {
        lock_guard<mutex> lck(mu);
        cout<<"Thread "<<this_thread::get_id()<<" sum="<<sum << endl;
    }
}

int main()
{
    vector<thread> th_v;            //存储线程的容器
    for(int i = 0;i < 10;++i)       //创建10个线程
    {
        th_v.push_back(thread(thread_func));
    }
    for(auto &th : th_v)
    {
        th.join();                  // 等待所有线程完成
    }
    cout<<"Main Thread "<<this_thread::get_id()<<" sum="<<sum <<endl;
}
```

程序输出

```text
Thread 2 sum=499999500000
Thread 3 sum=499999500000
Thread 4 sum=499999500000
Thread 5 sum=499999500000
Thread 6 sum=499999500000
Thread 7 sum=499999500000
Thread 8 sum=499999500000
Thread 9 sum=499999500000
Thread 10 sum=499999500000
Thread 11 sum=499999500000
Main Thread 1 sum=0
```

例子中的线程创建方式、互斥量等会在后面小节中介绍。

## 28.无限制的联合体（Unrestricted Unions）

`C++11`对联合体的限制放宽了限制，使其可以容纳任何非引用类型。

* 使用示例1

```cpp
#include <iostream>
#include <vector>

using namespace std;

union U
{
    int a;
    vector<int> b;          // 在C++11前不允许
};

int main()
{
    cout<<sizeof(U)<<endl;
    cout<<sizeof(vector<int>)<<endl;
}
```

程序输出

```text
12
12
```

此程序在不同编译器输出可能不同（取决于vector的实现以及指针的长度）。

## 29.类型特性（New character types）

`C++11`新增了很多类型，为编程提供便利，列举如下（仅列举了一些重要的类型，具体请查看[n1836.pdf](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1836.pdf)）：

 * 引用包装器： (\<functional>)
   * reference_wrapper
 * 智能指针：(\<memory>)
   * shared_ptr 
   * weak_ptr
   * enable_shared_from_this
 * 函数对象：(\<functional>) 
   * result_of
   * mem_fn
   * bind
   * placeholders
   * funcntion
 * 元编程与类型萃取(\<type_traits>)
   * is_xxx (用于判断某个类型或某几个类型是否满足某条件，如`is_pod<T>`用于判断`T`是否为`POD`类型，`is_same<A, B>`用于判断`A`和`B`是否为同一类型，该类有`value`数据成员，可用于编译期判断)。
   * add_xxx/remove_xxx（为类型添加某属性或删除某属性，如`add_const`，`remove_const`）
 * 数字设备(\<random>、\<cmath>)  
   * 随机数
 * 容器(\<tuple>、\<utility>、\<array>、\<functional>、\<unordered_set>、\<unordered_map>)
   * tuple
   * array
   * hash
   * unordered_set
   * unordered_map
   * unordered_multiset
   * unordered_multimap
 * 正则表达式(\<regex>)
   * regex
 * C兼容(\<complex>、\<cctype>、\<cfenv>、\<cfloat>、\<ios>、\<cinttypes>、\<climits>、\<locale>、\<cmath>、\<cstdarg>、\<cstdbool>、\<cstdint>、\<cstdio>、\<cstdlib>、\<ctgmath>、\<ctime>、\<cwchar>、\<cwctype>)

* 使用示例1（`ref`）

```cpp
#include <iostream>
#include <functional>

using namespace std;

template<typename T>
void func(T a)
{
    ++a;
}

int main()
{
    int a = 0, b = 0;
    func(a);                        // a以值类型传递，调用func后值不变
    cout << "a = " << a << endl;
    func(ref(b));                   // b以引用类型传递，调用后+1
    cout << "b = " << b << endl;
}
```

程序输出

```text
a = 0
b = 1
```

`ref`被定义为

```cpp
template< class T >
std::reference_wrapper<T> ref(T& t) noexcept;

template< class T >
std::reference_wrapper<T> ref( std::reference_wrapper<T> t ) noexcept;

template <class T>
void ref(const T&&) = delete;

template< class T >
std::reference_wrapper<const T> cref( const T& t ) noexcept;

template< class T >
std::reference_wrapper<const T> cref( std::reference_wrapper<T> t ) noexcept;

template <class T>
void cref(const T&&) = delete;
```

`ref`将参数转换为引用类型。

* 使用示例2（shared_ptr）

```cpp
#include <memory>
#include <iostream>

using namespace std;

class A
{
    private:
    int d;
    public:
    A(int t): d(t)
    {
        cout << d << "A()" << endl;
    }
    ~A()
    {
        cout << d <<"~A()"<<endl;
    }
};

int main()
{
    shared_ptr<A> pa(new A(1));        //第一种使用方式，使用new出来的对象构造
    {
        auto pb = make_shared<A>(2);        //第二种使用方式，使用make_shared辅助函数
    }
    cout << "end" << endl;
}
```

程序输出

```text
1A()
2A()
2~A()
end
1~A()
```

从输出可以看到，智能智能在结束作用域使会自动释放管理的内存。

* 使用示例3(weak_ptr)

```cpp
#include <iostream>
#include <memory>

using namespace std;

class B;

class A
{
    private:
    shared_ptr<B> b_ptr;
    public:
    A()
    {
        cout << "A()" << endl;
    }
    ~A()
    {
        cout<<"~A()"<<endl;
    }
    void set_ptr(shared_ptr<B> pb)
    {
        b_ptr = pb;
    }
};

class B
{
    private:
    shared_ptr<A> a_ptr;
    public:
    B()
    {
        cout << "B()" << endl;
    }
    ~B()
    {
        cout<<"~B()"<<endl;
    }
    void set_ptr(shared_ptr<A> pa)
    {
        a_ptr = pa;
    }
};

int main()
{
    auto sa = make_shared<A>();
    auto sb = make_shared<B>();
    // 让这两个对象互相持有对方指针
    sa->set_ptr(sb);
    sb->set_ptr(sa);
}
```

程序输出

```text
A()
B()
```

**这是一个错误的示例**，例子中，对象`sa`和`sb`分别持有对方指针，导致`sa`在作用域结束准备释放时，发现`sb`持有自身指针（引用计数不为0），所以不会释放，`sb`同理。接下来就用`weak_ptr`解决这种问题。

* 使用示例4（weak_ptr）

```cpp
#include <iostream>
#include <memory>

using namespace std;

class B;

class A
{
    private:
    shared_ptr<B> b_ptr;
    public:
    A()
    {
        cout << "A()" << endl;
    }
    ~A()
    {
        cout<<"~A()"<<endl;
    }
    void set_ptr(shared_ptr<B> pb)
    {
        b_ptr = pb;
    }
};

class B
{
    private:
    weak_ptr<A> a_ptr;
    public:
    B()
    {
        cout << "B()" << endl;
    }
    ~B()
    {
        cout<<"~B()"<<endl;
    }
    void set_ptr(shared_ptr<A> pa)
    {
        a_ptr = pa;
    }
};

int main()
{
    auto sa = make_shared<A>();
    auto sb = make_shared<B>();
    // 让这两个对象互相持有对方指针
    sa->set_ptr(sb);
    sb->set_ptr(sa);
}
```

程序输出

```text
A()
B()
~A()
~B()
```

这个例子与上一个例子唯一不同点就是`B`中对`A`的指针保存由`shared_ptr`变为`weak_ptr`，`weak_ptr`保存指针时，不会增加引用计数，所以对象`sa`在结束其作用域时，其引用计数为0，可正常释放，`sa`释放后，`sb`的引用计数为0，`sb`也会得到释放。（可查看析构函数的调用顺序）

因为`weak_ptr`不会增加引用计数，所以`weak_ptr`的指针并不是一定有效的（可能被其他对象释放了），所以`weak_ptr`不能直接解引用，而是要通过`lock()`函数获取到`shared_ptr`（如果被释放，返回`nullptr`，使用时需要判断）。

* 使用示例5（shared_from_this）

```cpp
#include <memory>
#include <iostream>

using namespace std;

class A;

void func(shared_ptr<A> pa)
{
    cout << "func" << endl;
}

class A : public enable_shared_from_this<A>      // 继承enable_share_from_this<A>，使类A可以使用shared_from_this
{
    public:
    void mem_func()
    {
        func(shared_from_this());               // shared_from_this 返回自身对象的一个shared_ptr，与该对象其他的指针共享引用计数
    }
};

int main()
{
    auto pa = make_shared<A>();
    pa->mem_func();
}
```

程序输出

```text
func
```

上例中，对象 `pa`需要产生一个指向自己的`shared_ptr`来调用`func`，因为`shared_ptr`必须是在堆上分配内存，所以不能`shared_ptr<A> pthis = this;`这样会引起引用计数为0时尝试释放栈内存的致命错误。而也不能使用`shared_ptr<A> pthis = new A();`此时的`pthis`不再是指向自身的指针了。

所以需要一个可以生成指向自身的`shared_ptr`的方法。`shared_from_this`的作用正是这个。

要使用`shared_from_this`，要使要管理自身的类继承自`enable_shared_from_this<A>`，这时会为类`A`生成`shared_from_this`成员函数，该函数生成管理自身的`shared_ptr`。

*注意*：调用`shared_from_this`的对象必须为被其他`shared_ptr`管理的对象，以下写法是错误的：

```cpp
A a;
auto pa = a.shared_from_this(); //a不是堆内存
A *b = new A();
auto pb = b->shared_from_this();    //b没有被其他shared_ptr管理
```

* 使用示例6（result_of）

```cpp
#include <type_traits>
#include <iostream>
 
struct S {
    double operator()(char, int&);
    float operator()(int) { return 1.0;}
};
 
template<class T>
typename std::result_of<T(int)>::type f(T& t)
{
    std::cout << "overload of f for callable T\n";
    return t(0);
}
 
template<class T, class U>
int f(U u)
{
    std::cout << "overload of f for non-callable T\n";
    return u;
}
 
int main()
{
    // 以 char 和 int 参数调用 S 的结果是 double
    std::result_of<S(char, int&)>::type d = 3.14; // d 拥有 double 类型
    static_assert(std::is_same<decltype(d), double>::value, "");
 
    // 以 int 参数调用 S 的结果是 float
    std::result_of<S(int)>::type x = 3.14; // x 拥有 float 累心
    static_assert(std::is_same<decltype(x), float>::value, "");
 
    // result_of 能以指向成员函数的指针以如下方式使用
    struct C { double Func(char, int&); };
    std::result_of<decltype(&C::Func)(C, char, int&)>::type g = 3.14;
    static_assert(std::is_same<decltype(g), double>::value, "");
 
    f<C>(1); // C++11 中可能编译失败； C++14 中调用不可调用重载
    S s;
    f(s);
}
```

程序输出

```text
overload of f for non-callable T
overload of f for callable T
```

`result_of`用于在编译期推导可调用类型、到函数引用或到可调用类型的引用的返回类型。

*注意：* `result_of`在`C++17`里被`invoke_result`替代。具体参考[cppreference.com](http://en.cppreference.com/w/cpp/types/result_of)

* 使用示例7（mem_fn）

```cpp
#include <functional>
#include <iostream>
 
struct Foo {
    void display_greeting() {
        std::cout << "Hello, world.\n";
    }
    void display_number(int i) {
        std::cout << "number: " << i << '\n';
    }
    int data = 7;
};
 
int main() {
    Foo f;
 
    auto greet = std::mem_fn(&Foo::display_greeting);
    greet(f);
 
    auto print_num = std::mem_fn(&Foo::display_number);
    print_num(f, 42);
 
    auto access_data = std::mem_fn(&Foo::data);
    std::cout << "data: " << access_data(f) << '\n';
}
```

程序输出

```text
Hello, world.
number: 42
data: 7
```

函数模板 `std::mem_fn` 生成指向成员指针的包装对象，它可以存储、复制及调用指向成员指针。到对象的引用和指针（含智能指针）可在调用 `std::mem_fn` 时使用。 

* 使用示例8（bind, placeholders）

```cpp
#include <random>
#include <iostream>
#include <memory>
#include <functional>
 
void f(int n1, int n2, int n3, const int& n4, int n5)
{
    std::cout << n1 << ' ' << n2 << ' ' << n3 << ' ' << n4 << ' ' << n5 << '\n';
}
 
int g(int n1)
{
    return n1;
}
 
struct Foo {
    void print_sum(int n1, int n2)
    {
        std::cout << n1+n2 << '\n';
    }
    int data = 10;
};
 
int main()
{
    using namespace std::placeholders;  // 对于 _1, _2, _3...
 
    // 演示参数重排序和按引用传递
    int n = 7;
    // （ _1 与 _2 来自 std::placeholders ，并表示将来会传递给 f1 的参数）
    auto f1 = std::bind(f, _2, _1, 42, std::cref(n), n);
    n = 10;
    f1(1, 2, 1001); // 1 为 _1 所绑定， 2 为 _2 所绑定，不使用 1001
                    // 进行到 f(2, 1, 42, n, 7) 的调用
 
    // 嵌套 bind 子表达式共享占位符
    auto f2 = std::bind(f, _3, std::bind(g, _3), _3, 4, 5);
    f2(10, 11, 12); // 进行到 f(12, g(12), 12, 4, 5); 的调用
 
    // 常见使用情况：以分布绑定 RNG
    std::default_random_engine e;
    std::uniform_int_distribution<> d(0, 10);
    std::function<int()> rnd = std::bind(d, e); // e 的一个副本存储于 rnd
    for(int n=0; n<10; ++n)
        std::cout << rnd() << ' ';
    std::cout << '\n';
 
    // 绑定指向成员函数指针
    Foo foo;
    auto f3 = std::bind(&Foo::print_sum, &foo, 95, _1);
    f3(5);
 
    // 绑定指向数据成员指针
    auto f4 = std::bind(&Foo::data, _1);
    std::cout << f4(foo) << '\n';
 
    // 智能指针亦能用于调用被引用对象的成员
    std::cout << f4(std::make_shared<Foo>(foo)) << '\n'
              << f4(std::make_unique<Foo>(foo)) << '\n';
}
```

程序输出：

```text
2 1 42 10 7
12 12 12 4 5
0 1 8 5 5 2 0 7 7 10
100
10
10
10
```

`bind(f, args ...)`函数模板 `bind` 生成转发调用包装器。调用此包装器等价于以一些绑定到 `args` 的参数调用`f` 。 

`std::placeholders` 命名空间含有占位对象 `[_1, . . . _N]` ，其中 `N` 是实现定义的最大数字。

于 `std::bind` 表达式用作参数时，占位符对象被存储于生成的函数对象，而以未绑定参数调用函数对象时，每个占位符 `_N` 被对应的第 `N` 个未绑定参数替换。(没有`_0`)

每个占位符如同以 `extern /*unspecified*/ _1;` 声明。

鼓励实现如同以 `inline constexpr /*unspecified*/ _1;` 声明占位符，尽管标准仍然允许以 `extern /*unspecified*/ _1;` 声明它们。

占位符对象的类型可默认构造 (DefaultConstructible) 且可复制构造 (CopyConstructible) ，其默认复制/移动构造函数不抛异常，且对于任何占位符 `_N` ，类型 `std::is_placeholder<decltype(_N)>` 有定义且从 `std::integral_constant<int, N>` 导出。

* 使用示例9（function）

```cpp
#include <functional>
#include <iostream>
 
struct Foo {
    Foo(int num) : num_(num) {}
    void print_add(int i) const { std::cout << num_+i << '\n'; }
    int num_;
};
 
void print_num(int i)
{
    std::cout << i << '\n';
}
 
struct PrintNum {
    void operator()(int i) const
    {
        std::cout << i << '\n';
    }
};
 
int main()
{
    // 存储自由函数
    std::function<void(int)> f_display = print_num;
    f_display(-9);
 
    // 存储 lambda
    std::function<void()> f_display_42 = []() { print_num(42); };
    f_display_42();
 
    // 存储到 std::bind 调用的结果
    std::function<void()> f_display_31337 = std::bind(print_num, 31337);
    f_display_31337();
 
    // 存储到成员函数的调用
    std::function<void(const Foo&, int)> f_add_display = &Foo::print_add;
    const Foo foo(314159);
    f_add_display(foo, 1);
    f_add_display(314159, 1);
 
    // 存储到数据成员访问器的调用
    std::function<int(Foo const&)> f_num = &Foo::num_;
    std::cout << "num_: " << f_num(foo) << '\n';
 
    // 存储到成员函数及对象的调用
    using std::placeholders::_1;
    std::function<void(int)> f_add_display2 = std::bind( &Foo::print_add, foo, _1 );
    f_add_display2(2);
 
    // 存储到成员函数和对象指针的调用
    std::function<void(int)> f_add_display3 = std::bind( &Foo::print_add, &foo, _1 );
    f_add_display3(3);
 
    // 存储到函数对象的调用
    std::function<void(int)> f_display_obj = PrintNum();
    f_display_obj(18);
}
```

程序输出

```text
-9
42
31337
314160
314160
num_: 314159
314161
314162
18
```

类模板 `std::function` 是通用多态函数封装器。 `std::function` 的实例能存储、复制及调用任何可调用 (Callable) 目标——函数、 `lambda` 表达式、 `bind` 表达式或其他函数对象，还有指向成员函数指针和指向数据成员指针。

存储的可调用对象被称为 `std::function` 的目标。若 `std::function` 不含目标，则称它为空。调用空 `std::function` 的目标导致抛出 `std::bad_function_call` 异常。

`std::function` 满足可复制构造 (CopyConstructible) 和可复制赋值 (CopyAssignable) 。 

* 使用示例10（random）

```cpp
#include <iostream>
#include <iomanip>
#include <string>
#include <map>
#include <random>
#include <cmath>
 
int main()
{
    // 以随机值播种，若可能
    std::random_device r;
 
    // 选择 1 与 6 间的随机数
    std::default_random_engine e1(r());
    std::uniform_int_distribution<int> uniform_dist(1, 6);
    int mean = uniform_dist(e1);
    std::cout << "Randomly-chosen mean: " << mean << '\n';
 
    // 生成围绕平均值的正态分布
    std::seed_seq seed2{r(), r(), r(), r(), r(), r(), r(), r()}; 
    std::mt19937 e2(seed2);
    std::normal_distribution<> normal_dist(mean, 2);
 
    std::map<int, int> hist;
    for (int n = 0; n < 10000; ++n) {
        ++hist[std::round(normal_dist(e2))];
    }
    std::cout << "Normal distribution around " << mean << ":\n";
    for (auto p : hist) {
        std::cout << std::fixed << std::setprecision(1) << std::setw(2)
                  << p.first << ' ' << std::string(p.second/200, '*') << '\n';
    }
}
```

可能输出：

```text
-6
-5
-4
-3 *
-2 ***
-1 *****
 0 *********
 1 *********
 2 ********
 3 ******
 4 ***
 5 *
 6
 7
 8
 9
```

随机数库提供生成随机和伪随机数的类。这些类包括：

    随机数引擎（生成拥有均匀分布整数序列的伪随机数生成器，和若可用则如此的真随机数生成器）。
    随机数分布（例如均匀、正态或泊松分布），它们将随机数引擎的输出转换为各种统计分布。 

引擎和分布被设计为相互使用以生成随机值。所有引擎都可以指定地播种、序列化和反序列化，以用于可重复的模拟器。 

*随机数引擎：*

随机数引擎以种子数据为熵源生成伪随机数。数种不同类的伪随机数生成算法实现为能定制的模板。

选择使用何种引擎涉及到多次权衡：线性同余引擎一般很快，并对状态的存储要求非常小。延迟斐波那契生成器在无先进算术指令集的处理器上非常快，但状态存储较为庞大，有时有不太想要的光谱特性。梅森缠绕器较慢且拥有较大的状态存储要求，但只要有正确的参数，就会有最长的的不可重复序列，且拥有最想要的光谱特性（对于给定的想要的定义）。

*随机数引擎适配器：*

随机数引擎适配器生成以另一随机数引擎为熵源的伪随机数。它们通常用于改换底层引擎的光谱特性。 

*预定义随机数生成器：*

定义了数个特别的流行算法。 

*非确定随机数：*

`std::random_device` 是非确定的均匀随机数生成器，尽管若不支持非确定随机数生成，则允许实现用伪随机数引擎实现 `std::random_device` 。

*随机数分布：*

随机数分布后处理随机数的发生结果，以使得输出结果按照定义的统计概率密度函数分布。

* 使用示例11（tuple）

```cpp
#include <tuple>
#include <iostream>
#include <string>
#include <stdexcept>
 
std::tuple<double, char, std::string> get_student(int id)
{
    if (id == 0) return std::make_tuple(3.8, 'A', "Lisa Simpson");
    if (id == 1) return std::make_tuple(2.9, 'C', "Milhouse Van Houten");
    if (id == 2) return std::make_tuple(1.7, 'D', "Ralph Wiggum");
    throw std::invalid_argument("id");
}
 
int main()
{
    auto student0 = get_student(0);
    std::cout << "ID: 0, "
              << "GPA: " << std::get<0>(student0) << ", "
              << "grade: " << std::get<1>(student0) << ", "
              << "name: " << std::get<2>(student0) << '\n';
 
    double gpa1;
    char grade1;
    std::string name1;
    std::tie(gpa1, grade1, name1) = get_student(1);
    std::cout << "ID: 1, "
              << "GPA: " << gpa1 << ", "
              << "grade: " << grade1 << ", "
              << "name: " << name1 << '\n';
}
```

程序输出

```text
ID: 0, GPA: 3.8, grade: A, name: Lisa Simpson
ID: 1, GPA: 2.9, grade: C, name: Milhouse Van Houten
```

类模板 `std::tuple`（元组） 是固定大小的异类值汇集。它是 `std::pair` 的通用化。

* 使用示例12（array）

```cpp
#include <string>
#include <iterator>
#include <iostream>
#include <algorithm>
#include <array>
 
int main()
{
    // 用聚合初始化构造
    std::array<int, 3> a1{ {1, 2, 3} }; // C++11 中要求双花括号（ C++14 中不要求）
    std::array<int, 3> a2 = {1, 2, 3};  // 决不要求在 = 后
    std::array<std::string, 2> a3 = { std::string("a"), "b" };
 
    // 支持容器操作
    std::sort(a1.begin(), a1.end());
    std::reverse_copy(a2.begin(), a2.end(), 
                      std::ostream_iterator<int>(std::cout, " "));
 
    std::cout << '\n';
 
    // 支持带范围 for 循环
    for(const auto& s: a3)
        std::cout << s << ' ';
}
```

程序输出

```text
3 2 1
a b
```

`std::array` 是封装固定大小数组的容器。

此容器是一个聚合类型，其语义等同于保有一个 C 风格数组 `T[N]` 作为其唯一非静态数据成员的结构体。不同于 C 风格数组，它不会自动退化成 `T*` 。作为聚合类型，它能聚合初始化，只要有至多 `N` 个能转换成 `T` 的初始化器： `std::array<int, 3> a = {1,2,3};`。

该结构体结合了 C 风格数组的性能和可访问性和容器的优点，譬如知晓其大小、支持赋值、随机访问等。

`std::array` 满足容器 (Container) 和可逆容器 (ReversibleContainer) 的要求，除了默认构造的 `array` 是非空的，及交换的复杂度是线性，它满足相接容器 (ContiguousContainer) 的要求并 (C++17 起)部分满足顺序容器 (SequenceContainer) 的要求。

一种特殊情况是 `array(N == 0)`。该情况下， `array.begin() == array.end()` ，并拥有某个唯一值。在零长 `array` 上调用 `front()` 或 `back()` 的效应是未定义的。

`array` 亦可用作拥有 `N` 个同类型元素的元组。

* 使用示例13（hash）

```cpp
#include <iostream>
#include <iomanip>
#include <functional>
#include <string>
#include <unordered_set>
 
struct S {
    std::string first_name;
    std::string last_name;
};
bool operator==(const S& lhs, const S& rhs) {
    return lhs.first_name == rhs.first_name && lhs.last_name == rhs.last_name;
}
 
// 自定义哈希能是独立函数对象：
struct MyHash
{
    std::size_t operator()(S const& s) const 
    {
        std::size_t h1 = std::hash<std::string>{}(s.first_name);
        std::size_t h2 = std::hash<std::string>{}(s.last_name);
        return h1 ^ (h2 << 1); // 或使用 boost::hash_combine （见讨论）
    }
};
 
// std::hash 的自定义特化能注入 namespace std
namespace std
{
    template<> struct hash<S>
    {
        typedef S argument_type;
        typedef std::size_t result_type;
        result_type operator()(argument_type const& s) const
        {
            result_type const h1 ( std::hash<std::string>{}(s.first_name) );
            result_type const h2 ( std::hash<std::string>{}(s.last_name) );
            return h1 ^ (h2 << 1); // 或使用 boost::hash_combine （见讨论）
        }
    };
}
 
int main()
{
 
    std::string str = "Meet the new boss...";
    std::size_t str_hash = std::hash<std::string>{}(str);
    std::cout << "hash(" << std::quoted(str) << ") = " << str_hash << '\n';
 
    S obj = { "Hubert", "Farnsworth"};
    // 使用独立的函数对象
    std::cout << "hash(" << std::quoted(obj.first_name) << ',' 
               << std::quoted(obj.last_name) << ") = "
               << MyHash{}(obj) << " (using MyHash)\n                           or "
               << std::hash<S>{}(obj) << " (using std::hash) " << '\n';
 
    // 自定义哈希令在无序容器中使用自定义类型可行
    // 此示例将使用注入的 std::hash 特化，
    // 若要使用 MyHash 替代，则将其作为第二模板参数传递
    std::unordered_set<S> names = {obj, {"Bender", "Rodriguez"}, {"Leela", "Turanga"} };
    for(auto& s: names)
        std::cout << std::quoted(s.first_name) << ' ' << std::quoted(s.last_name) << '\n';
}
```

程序输出

```text
hash("Meet the new boss...") = 2535640445
hash("Hubert","Farnsworth") = 3604770409 (using MyHash)
                           or 3604770409 (using std::hash)
"Leela" "Turanga"
"Bender" "Rodriguez"
"Hubert" "Farnsworth"
```

`hash` 函数模板的启用的特化 (`C++17` 起)定义一个实现哈希函数的函数对象。此函数对象的实例满足哈希 (Hash) 。特别是它们定义满足下列条件的 `operator()` ：

    1.接收 Key 类型的单个参数
    2. 返回表示参数哈希值的 size_t 类型。
    3. 调用时不抛出异常。
    4. 对于二个相等的参数 k1 与 k2 ， std::hash<Key>()(k1) == std::hash<Key>()(k2) 。
    5. 对于二个相异而不相等的参数 k1 与 k2 ， std::hash<Key>()(k1) == std::hash<Key>()(k2) 的概率应非常小，接近 1.0/std::numeric_limits<size_t>::max() 。

库函数提供的 `hash` 的所有显式和部分特化可复制构造 (DefaultConstructible) 、可复制赋值 (CopyAssignable) 、可交换 (Swappable) 且可析构 (Destructible) 。用户提供的 `hash` 特化亦必须满足这些要求。

无序关联容器 `std::unordered_set` 、 `std::unordered_multiset` 、 `std::unordered_map` 、 `std::unordered_multimap` 以该模板 `std::hash` 的特化为默认哈希函数。
注意

实际的哈希函数是实现定义的，且不要求满足任何其他质量标准，除了上述指定者。值得注意的是，一些实现使用映射整数到其自身的平凡（自身）哈希函数。换言之，这些哈希函数设计为与无序关联容器一同工作，然而不是加密哈希之类的哈希函数。

哈希函数仅要求在程序的单次执行中对同样的输入返回同样的结果；这允许避免碰撞 DoS 攻击的加盐哈希。 	(`C++14` 起)

没有对 C 字符串的特化。 `std::hash<const char*>` 产生指针值（内存地址）的哈希，它不检验任何字符数组的内容。 

* 使用示例14（unordered_xxx）

```cpp
#include <unordered_map>
#include <unordered_set>
#include <iostream>
#include <iomanip>

using namespace std;

int main()
{
    unordered_set<int> un_set {10,3,2,5,9,7,0,9};   // 去除重复元素
    for( auto &p : un_set)
    {
        cout<<p<<endl;                              // 不会裴谞，也不会按照插入顺序输出，而是根据元素的hash
    }

    unordered_map<string, int> un_map {
        {"hello", 25},
        {"world", 56},
        {"lalala", 68},
        {"666", 77}
    };

    for(auto &p : un_map)
    {
        cout << "(" << quoted(p.first) << "," << p.second << ")" << endl;
    }
}
```

程序输出

```text
0
7
9
5
2
3
10
("666",77)
("lalala",68)
("world",56)
("hello",25)
```

无序关联容器是提供基于关键的快速对象查找的容器 (Container) 。最坏情况的复杂度为线性，但平均而言大多数操作更快。

无序关联容器为 `Key` 所参数化； `Hash` ，表现为 `Key` 上哈希函数的哈希 (Hash) 函数对象；而 `Pred` ，是求值 `Key` 间等价性的二元谓词 (BinaryPredicate) 。 `std::unordered_map` 与 `std::unordered_multimap` 也拥有关联到 `Key` 的被映射类型 `T` 。

若二个 `Key` 按照 `Pred` 比较相等，则 `Hash` 必须对两个关键返回相同值。

`std::unordered_map` 与 `std::unordered_set` 能容纳至多一个有给定关键的元素，而 `std::unordered_multiset` 与 `std::unordered_multimap` 能拥有带同一关键的多个元素（它们在迭代时必须相邻）。

对于 `std::unordered_set` 和 `std::unordered_multiset` ， `value_type` 与关键类型相同，而 `iterator` 和 `const_iterator` 都是常迭代器。对于 `std::unordered_map `与 `std::unordered_multimap` ， `value_type` 是 `std::pair<const Key, T>` 。

无序关联容器中的元素被组织到桶中，拥有相同哈希的关键将归结于相同的桶。桶数在容器大小增加时增加，以保持每个桶中的平均元素数在确定值之下。

重哈希非法化迭代器，并可能导致元素重排到不同的桶，但它不非法化到元素的引用。

无序关联容器满足具分配器容器 (AllocatorAwareContainer) 的要求。对于 `std::unordered_map` 与 `std::unordered_multimap` ，具分配器容器 (AllocatorAwareContainer) 中 `value_type` 的要求应用到 `key_type` 和 `mapped_type` （而非到 `value_type` ）。 

* 使用示例15（regex）

```cpp
#include <iostream>
#include <iterator>
#include <string>
#include <regex>
 
int main()
{
    std::string s = "Some people, when confronted with a problem, think "
        "\"I know, I'll use regular expressions.\" "
        "Now they have two problems.";
 
    std::regex self_regex("REGULAR EXPRESSIONS",
            std::regex_constants::ECMAScript | std::regex_constants::icase);
    if (std::regex_search(s, self_regex)) {
        std::cout << "Text contains the phrase 'regular expressions'\n";
    }
 
    std::regex word_regex("(\\S+)");
    auto words_begin = 
        std::sregex_iterator(s.begin(), s.end(), word_regex);
    auto words_end = std::sregex_iterator();
 
    std::cout << "Found "
              << std::distance(words_begin, words_end)
              << " words\n";
 
    const int N = 6;
    std::cout << "Words longer than " << N << " characters:\n";
    for (std::sregex_iterator i = words_begin; i != words_end; ++i) {
        std::smatch match = *i;
        std::string match_str = match.str();
        if (match_str.size() > N) {
            std::cout << "  " << match_str << '\n';
        }
    }
 
    std::regex long_word_regex("(\\w{7,})");
    std::string new_s = std::regex_replace(s, long_word_regex, "[$&]");
    std::cout << new_s << '\n';
}
```

程序输出：

```text
Text contains the phrase 'regular expressions'
Found 19 words
Words longer than 6 characters:
  people,
  confronted
  problem,
  regular
  expressions."
  problems.
Some people, when [confronted] with a [problem], think "I know, I'll use [regular] [expressions]." Now they have two [problems].
```

正则表达式库提供表示正则表达式的类，正则表达式是一种用于在字符串中匹配模式的微型语言。下列数种对象上的操作能刻画几乎所有带正则表达式的操作：

    目标序列。为模式而搜索到的字符序列。这可以是二个迭代器所指定的范围、空终止字符串或一个 std::string 。 

    模式。这是正则表达式自身。它确定构成匹配者。它是从带特定语法的字符串构成的 std::basic_regex 类型对象。

    匹配的数组。关于匹配的信息可作为 std::match_results 类型对象获取。 

    替换字符串。这是确定如何替换匹配的字符串，受支持的语法变体的描述见 match_flag_type 。 

## 30.边长模板（Variadic Templates）

`C++11`对边长模板提供了支持。使用`...`来表示参数包（也用`...`来解包）。`sizeof...`运算符来计算参数包中参数个数。

详情见[n2242](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2242.pdf)与[n2555](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2555.pdf)

```cpp
#include <iostream>

using namespace std;

template<typename T>
void func_helper(T t)                   // 递归结束条件
{
    cout<<t<<endl;
}

template<typename T, typename ... Args>
void func_helper(T t, Args ... t2)
{
    func_helper(t2...);                 // 参数解包
    cout<<t<<endl;                      // 逆序打印出参数
}

template<typename ... Args>
void func(Args ... t)
{
    cout << sizeof...(t)<<endl;         // sizeof... 运算符计算可变参数数量
    func_helper(t ...);
}

int main()
{
    func(5,3.5,"hello",'t');
}
```

程序输出

```text
4
t
hello
3.5
5
```

## 31.范围for循环（range for）

`C++11`提供基于范围的`for`循环。可以大大简化遍历容器的代码。

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    vector<int> arr {1,2,3,4,5,6};
    for(auto &p : arr)          // 元素以引用方式取出
    {
        cout << p <<endl;
        ++p;                    // 可以改变原容器中的元素
    }
    cout << "------" << endl;
    for(auto &p : arr)
    {
        cout << p <<endl;
    }
}
```

程序输出

```text
1
2
3
4
5
6
------
2
3
4
5
6
7
```

## 32.override和final

在成员函数声明或定义中， `override` 确保该函数为虚并覆写来自基类的虚函数。若此非真则程序为病式（生成编译错误）。

`override` 是在成员函数声明器后使用时拥有特殊含义的标识符：其他情况下它不是保留的关键词。

在虚函数声明或定义中使用时， `final` 确保函数为虚且不可被派生类覆写。若这么做则会发生编译错误。

在类定义中使用时， `final` 指定此类不可出现于另一类的定义的 `base-specifier-list` 中（换言之，不能派生它）。若这么做则生成编译时错误。 `final` 亦可用于联合体定义，此情况下它无效（除了 `std::is_final` 上的结果），因为不能派生联合体。

`final` 是在用于成员函数声明或类头部时有特殊含义的标识符。其他语境中它非保留而且可用于命名对象或函数。

* 使用示例1

```cpp
class A
{
    public:
    virtual void vfunc(int ) const
    {
    }
    virtual void vfunc2(int ) final
    {
    }
};

class B : public A
{
    public:
    // void vfunc(int ) override { }  // 编译错误，因为函数签名不一致，并非重写
    void vfunc(int) const override {}       // ok

    // void vfunc2(int ) override { } // 编译错误，无法重写final函数
};

int main()
{

}
```

## 33.属性指定符序列（attributes）

属性为实现定义的语言扩展，例如 GNU 与 IBM 语言扩展 `__attribute__((...))` 、微软语言扩展 `__declspec()` 等提供提供统一化的语法。

属性可用在 C++ 程序的几乎所有位置，而且可应用于几乎所有事物：到类型、变量、函数、名称、代码块、整个翻译单元，不过每个特定属性仅在实现所容许处合法： `[[likely]]` 可能是只能与 `if` ，而非与类声明一同使用的属性， `[[omp::parallel()]]` 可能是应用到代码块或 `for` 循环，而非到类型 `int` 等的属性。（注意此二属性是虚构示例，见后述的标准与一些非标准属性）

在声明中，属性可出现在被声明实体名称前或紧在其后，这些情况下它们被结合。多数其他情形中，属性仅应用于直接前附的实体。

`alignas` 指定符是属性序列指定符的一部分，尽管它拥有不同语法。它可出现于 `[[...]]` 属性出现处，并可与它们混合（假定用于容许 `alignas` 处）

二个连续的方括号记号（ `[[` ）只能出现于引入属性指定符处，或在属性参数内。

```cpp
void f() {
  int y[3];
  y[[] { return 0; }()] = 1;    // 错误
  int i [[cats::meow([[]])]]; // OK
}
```

除了下列的标准属性，实现还可以支持任意拥有实现定义行为的非标准属性。所有实现所未知的属性被忽略，且不产生错误。 (C++17 起)

* 使用示例1

```cpp
#include <cassert>
[[ noreturn ]] void f() {
  throw "error";
  // OK
}
 
[[ noreturn ]] void q(int i) {
  // 若以 <= 0 的参数调用则行为未定义
  if (i > 0) {
    throw "positive";
  }
}

void g(){}
void h(){}
void i(){}
 
void f(int n) {
  void g(), h(), i();
  switch (n) {
    case 1:
    case 2:
      g();
     [[fallthrough]];
    case 3: // 落下时无警告
      h();
    case 4: // 编译器可能对落下警告
      i();
      [[fallthrough]]; // 病态，不在 case 标号前
  }
}
 
struct [[nodiscard]] error_info { };
error_info enable_missile_safety_mode(){
    return error_info();
}
void launch_missiles(){};
void test_missiles() {
   enable_missile_safety_mode(); // 编译器可能对丢弃一个 nodiscard 值警告
   launch_missiles();
}
error_info& foo(){
    return *(new error_info);
};
void f1() {
    foo(); // 不以值返回 nodiscard 类型，无警告
} 
 
 
[[maybe_unused]] void f([[maybe_unused]] bool thing1,
                        [[maybe_unused]] bool thing2)
{
   [[maybe_unused]] bool b = thing1 && thing2;
   assert(b); // 在 release 模式， assert 会被编译去除，而 b 不被使用
              // 无警告，因为它声明带 [[maybe_unused]]
} // 不使用参数 thing1 与 thing2 ，无警告

int main(){}
```

## 34.引用限定符（ref-qualifiers）

`C++11`增加了引用限定符，可以限制调用者的类型（左值还是右值）

* 使用示例1

```cpp
#include <iostream>
using namespace std;

class A{
    public:
    void func() &       // 只有左值可以调用
    {
        cout<<"func()&"<<endl;
    }
    void func()&&       // 只有右值可以调用
    {
        cout<<"func()&&"<<endl;
    }
};

A make_a()
{
    return A();
}

int main()
{
    A a;
    a.func();
    make_a().func();
    move(a).func();
}
```

程序输出

```text
func()&
func()&&
func()&&
```

## 35.非静态成员的类内初始化（Non-static data member initializers）

`C++11`让非静态成员可以在定义的时候初始化。不必在构造函数中初始化。

* 使用示例1

```cpp
#include <iostream>
#include <string>

using namespace std;

class A
{
    private:
    int a = 0;
    string  b = "str";
    public:
    A(): a(5) {}        // 构造函数会在类内初始化之后进行，所以a的值会变成5
    void print()
    {
        cout<<"a="<<a<<endl;
        cout<<"b="<<b<<endl;
    }
};

int main()
{
    A a;
    a.print();
}
```

程序输出

```text
a=5
b=str
```

## 36.noxcept限定符

`C++11`增加`noxcept`限定符，使编译器在编译过程中实现不带`throw()`的运行期优化。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

void func() noexcept
{
    throw 5;        // noexcept不能保证一定不抛出异常
}

int main()
{
    try
    {
        func();
    }
    catch(int e)     // noexcept函数抛出的异常无法被catch到
    {
        cout<<"Exception:"<<e<<endl;
    }
}
```

此程序会在运行时崩溃。

## 37.金额、时间及十六进制浮点 I/O 操纵符（new io）

`C++11`增加了一组用于输入输出货币金额、时间的函数。

* 使用示例1

```cpp
#include <iostream>
#include <iomanip>
#include <ctime>
 
int main()
{
    std::time_t t = std::time(nullptr);
    std::tm tm = *std::localtime(&t);
    std::cout << "ru_RU: " << std::put_time(&tm, "%c %Z") << '\n';
}
```

程序输出

```text
02/11/18 13:41:57 中国标准时间
```

增加的函数包括：

`get_money` 剖析货币值 

`put_money` 格式化并输出货币值 

`get_time` 剖析指定格式的日期/时间值 

`put_time` 按照指定格式格式化并输出日期/时间值 

`quoted` 插入和读取带有内嵌空格的被引号括起来的字符串 

## 38.二进制字面量（binary­ literal）

`C++14`增加了二进制字面量。以`0b`或`0B`开头的字面量是二进制字面量。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout<<0b01001110<<endl;
}
```

程序输出

```text
78
```

## 39.变量模板（Variable Templates）

`C++14`允许定义变量模板，可以使程序更易于理解。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

template<typename T>
constexpr T var(5.94);

void func(int a)
{
    cout<<"int"<<endl;
}

void func(double a)
{
    cout<<"double"<<endl;
}

int main()
{
    func(var<int>);
    func(var<double>);
}
```

程序输出

```text
int
double
```

## 40.成员初始化器与聚合体（Member initializers and aggregates）

`C++14`提供了更加便利的聚合体初始化方法。

聚合体是一个数组或类，没有用户提供的构造函数，没有非静态数据成员，没有私有或受保护的非静态数据成员，没有基类，也没有虚函数。

如果列表中的初始化子句比集合中的成员少，那么没有显式初始化的每个成员都应该从括号\{\}或等初始化器中初始化，如果没有括号或等初始化器，则从一个空的初始化器列表中进行初始化。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

struct X { int i, j, k = 42; };

int main()
{
    X a[] = { 1, 2, 3, 4, 5, 6 };
    X b[2] = { { 1, 2, 3 }, { 4, 5, 6 } };
    for(auto p = begin(a); p!=end(a);++p)
    {
        cout<<"a_i->"<<p->i<<endl;
        cout<<"a_j->"<<p->j<<endl;
        cout<<"a_k->"<<p->k<<endl;
    }
    for(auto p = begin(b); p!=end(b);++p)
    {
        cout<<"b_i->"<<p->i<<endl;
        cout<<"b_j->"<<p->j<<endl;
        cout<<"b_k->"<<p->k<<endl;
    }
}
```

程序输出

```text
a_i->1
a_j->2
a_k->3
a_i->4
a_j->5
a_k->6
b_i->1
b_j->2
b_k->3
b_i->4
b_j->5
b_k->6
```

## 41.单引号作为数位分隔符（Digit Separator）

`C++14`新增了使用单引号作为数字分隔符，使长数字字面量更加清晰。

* 使用示例1

```cpp
#include <iostream>

using namespace std;

int main()
{
    int a = 10;
    int b = 1'234'567;
    cout<<a+b<<endl;
}
```

程序输出

```text
1234577
```

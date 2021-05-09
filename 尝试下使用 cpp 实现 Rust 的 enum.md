# 尝试下使用 cpp 实现 Rust 的 enum

长篇代码警告：本文建议边看边敲（`Ctrl C V`），否则可能会犯困~~~

本文中代码运行需要`C++20`支持，相信幼儿园小班就学过`docker`的你肯定能搞定编译环境的啦，么么哒~~~

## 一个小背景，Rust 中伪装成 enum 的 class

Rust 中的 enum 类型很强大，如下：

```rust
fn main() {
    enum Book {
        Papery {
            name: String,
            price: u32,
            author: String,
        },
        Electronic {
            name: String,
            url: String,
            author: String,
            filetype: String,
        },
    }

    let book = Book::Papery {
        name: String::from("C++变成死相"),
        price: 30,
        author: String::from("Zhang San"),
    };
    let ebook = Book::Electronic {
        name: String::from("加瓦入门到精通"),
        url: String::from("xxxx.xxx.xxx"),
        author: String::from("Li Si"),
        filetype: String::from("pdf"),
    };
    match book {
        Book::Papery {
            name,
            price,
            author,
        } => {
            println!(
                "Papery book name: {}, price:{} author: {}",
                name, price, author
            );
        }
        Book::Electronic {
            name,
            url,
            author,
            filetype,
        } => {
            println!(
                "E-book name: {} url:{} author: {} filetype:{}",
                name, url, author, filetype
            );
        }
    }

    match ebook {
        Book::Papery {
            name,
            price,
            author,
        } => {
            println!(
                "Papery book name: {}, price:{} author: {}",
                name, price, author
            );
        }
        Book::Electronic {
            name,
            url,
            author,
            filetype,
        } => {
            println!(
                "E-book name: {} url:{} author: {} filetype:{}",
                name, url, author, filetype
            );
        }
    }
}
```

如上所示， `Rust` 中的 `enum` 是可以带属性的，而且还可以带不止一种属性（这明明就是伪装成 `enum` 的 `class` 么~~~），使用 `match` 可以对 `enum` 进行匹配和提取（和 `C++` 的泛型以及类型萃取很像有木有？）， `enum` 还有一些其他的用法，这里先不考虑，仅仅考虑下用 `C++` 实现上面例子中对 `enum` 特性的展示。顺便复习下 `N` 久没用过的 `C++` ！（好像也就一两天？）

## 如何实现？先列个计划！

毛爷爷说过：“用百折不回的毅力，有计划地克服所有的困难。”

我们现在就要克服用 `C++` 实现 `Rust` 中 `enum` 的困难，首先，第一步当然是给我们的新类型起个名字啦，既然是要扩展 `enum` 类型，那么就叫 `enum_ex` 吧！反正起名一直是让程序员头疼的难题。有了名字，我们就要开始实现啦！

等等！说好的计划呢？

计划不是有么，分两步：1. 取名！ 2. 实现！ <_<

## 实现

### 第0步：搞一个类型来表示每个枚举项

从上面的例子可以看出， `Rust` 的每个枚举项除了自身的名字外，还有一大波属性（姑且称之为 `PropertyTypes` 吧），它自身的名字，更像是一个类型，这里就把它叫做 `IndexType` 吧。

一个类，包含一个索引类型和若干属性类型，用幼儿园中班就学过的可变参数模板就很容易实现，同时我们还可以顺便把它们萃取出来先放着，未雨绸缪嘛，以后总会用得着的。那么怎样在同一个对象里面表达多个类型呢？对！就是它，幼儿园小班就学过的 `tuple` 呀：

```cpp
#pragma once

#include <tuple>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = IndexType;
    using value_type = std::tuple<PropertyTypes...>;
};
```

很容易就写出这样的代码啦， `enum_ex` 瞬间具有了类型萃取能力，很方便有没有。

另外呢，为了防止一些用户丢一些带限定符呀引用呀等等等等乱七八糟的类型过来，我们对传入类型做一个简单的清理：

```cpp
#pragma once

#include <tuple>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = typename std::decay<IndexType>::type;
    using value_type = std::tuple<typename std::decay<PropertyTypes>::type...>;
};
```

众所周知， `std::decay`可以剥去类型的限定符外衣，让它赤裸裸地暴露在你面前。当然，如果你认为我上面这段代码有装逼嫌疑的话，也可以直接使用`std::decay_t`，不过由于我的`vscode`不太喜欢`std::xxxx_t`这种东西（没事就给我显示个红色波浪线），这个逼我暂时就继续装着吧。

### 第1步：开始构造一个 enum_ex

上面的步骤，我们构造出了一个个的枚举项，现在我们要用一群枚举项实例化一个枚举类，大致使用就是这样：

```cpp
struct Papery {};
struct Electronic {};

using Book = enum_ex<
    enum_item<Papery, std::string, int, std::string>,
    enum_item<Electronic, std::string, std::string, std::string, std::string>
>;
```

这样就和 Rust 有那么一丢丢像啦 ^_^

那么，如何实现这个呢？聪明的幼儿园小班小盆友一定想到啦，当然是我们的可变参数模板啦：

```cpp
template<typename ... EnumTypes>
class enum_ex{
};
```

But, But, But! 这里有个问题，如果用户瞎写：

```cpp
using Book = enum_ex<
    int, 
    double,
    enum_item<int, std::string>
>;
```

仿佛也可以编译通过呀，这可是个问题，那么有没有办法限定 `enum_ex` 的模板参数必须是 `enum_item` 的实例化类型呢？当然有啦，我 `C++` 这么能打，这都是小 `case` 啦，众所周知，在 `C++20` 前，可以通过`std::enable_if`来进行`SFINAE`，不过我们已经有了 `C++20`，他的`concept`天生就是干这事的，我们使用`concept`对类型限制如下：

```cpp
#pragma once

#include <concepts>
#include <tuple>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = typename std::decay<IndexType>::type;
    using value_type = std::tuple<typename std::decay<PropertyTypes>::type...>;
};

template <typename T>
struct is_enum_item : public std::false_type {
};

template <typename... Types>
struct is_enum_item<enum_item<Types...>> : public std::true_type {
};

template <typename T>
concept EnumTypeConcept = is_enum_item<T>::value;

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&& ... && true) 
class enum_ex {
    
};
```

上面这个代码，相信幼儿园小班的小盆友们一定都看懵逼了，所以这可能需要幼儿园中班的知识了，上面的这一坨代码涉及了`concept`、`模板类全特化`、折叠表达式（`fold expression`），不懂的小盆友赶紧去 [cppreference](https://en.cppreference.com)了解一下，否则会很印象进度的哦。

上面先使用`is_enum_item`将从`enum_item`实例化的类型摘出来，不是`enum_item`的走`false_type`那个实例化，是`enum_item`的走`true_type`那个实例化，然后把这个值绑定到`concept` `EnumTypeConcept`上。

在`enum_ex`实例化时，使用`EnumTypeConcept`对每个模板实参做校验，然后将所有的结果逻辑与连起来，形成最终校验。

其实还可以使用`template<EnumTypeConcept ... EnumTypes>`来校验，但是斯以为没有`requires`这种显式校验来的清晰。

### 第2步：如何从 IndexType 找到 enum_item

我们在`match`的时候，可以通过一个索引类型直接提取出对应的属性值，所以第一步要解决的问题就是怎样从`IndexType`找到对应的`enum_item`。

```cpp
template <typename IndexType, typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) struct find_enum_item {
    using type = nullptr_t;
};

template <typename IndexType, typename FirstType, typename... EnumTypes>
    requires (EnumTypeConcept<EnumTypes> && ... && EnumTypeConcept<FirstType>) struct find_enum_item<IndexType, FirstType, EnumTypes...> {
    using type = typename std::conditional<
        std::is_same<IndexType, typename FirstType::index_type>::value,
        FirstType,
        typename find_enum_item<IndexType, EnumTypes...>::type>::type;
};
```

一样的套路，使用`find_enum_item`找到与`enum_item`类型集合的`index_type`与给定`IndexType`一致的类型。

再对类型脱个衣服：

```cpp
template <typename IndexType, typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) struct find_enum_item {
    using type = nullptr_t;
};

template <typename IndexType, typename FirstType, typename... EnumTypes>
requires(EnumTypeConcept<typename std::decay<EnumTypes>::type>&&...&& EnumTypeConcept<typename std::decay<FirstType>::type>) struct find_enum_item<IndexType, FirstType, EnumTypes...> {
    using type = typename std::conditional<
        std::is_same<typename std::decay<IndexType>::type, typename FirstType::index_type>::value,
        typename std::decay<FirstType>::type,
        typename find_enum_item<IndexType, EnumTypes...>::type>::type;
};
```

写一段代测试一下：

```cpp
#include "enum_ex.h"

#include <string>

struct Papery {
};
struct Electronic {
};

int main()
{
    static_assert(std::is_same<
                      typename find_enum_item<
                          Papery,
                          enum_item<Papery, std::string, int, std::string>,
                          enum_item<Electronic, std::string, std::string, std::string, std::string>>::type,
                      enum_item<Papery, std::string, int, std::string>>::value,
        "type mismatch");
}
```

一波编译， 0 error(s), 0 warning(s) ,Beautiful！

### 第3步：构造一个 enum_ex

我们希望我们的`enum_ex`使用越简单越好，最好是可以这样：

```cpp
using Book = enum_ex<
    enum_item<Papery, std::string, int, std::string>,
    enum_item<Electronic, std::string, std::string, std::string, std::string>
>;

Book<Papery> book("C++变成死相", 30, "Zhang San");
```

不过很遗憾，这种玩法是不可能了，因为类型`Book`是已经实例化的类型了，没办法使用`<>`来创建对象了。不过我们可以退而求其次，使用一个`static`模板函数来创建对象。但是`static`模板函数的参数列表不好处理（不信你试试？），我们可以换用`static`成员变量，只不过他的类型是个仿函数。

那么问题又来了，函数签名应该长啥样呢？既然要创建`enum_ex`类型，那肯定是返回一个`enum_ex`啦，模板参数自然就是`IndexType`，形参类型也就是`PropertyTypes`啦。所以我们需要一个新的类型来构造这个函数类型：

```cpp

template <typename T>
struct is_tuple : std::false_type {
};

template <typename... Types>
struct is_tuple<std::tuple<Types...>> : public std::true_type {
};

template <typename T>
concept TupleTypeConcept = is_tuple<T>::value;

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires(TupleTypeConcept<TupleType>&&...&& EnumTypeConcept<EnumTypes>) struct gen_func_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires(true && ... && EnumTypeConcept<EnumTypes>) struct gen_func_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<enum_ex<EnumTypes...>(TupleTypes...)>;
};
```

这段代码很好理解，最前面那几坨就是加个`tuple`的`concept`以及类前置声明，可以忽略。后面的那两坨就是传入索引类型`IndexType`、`tuple`类型和枚举项的类型，生成一个以`tuple`的形参类型作为参数类型，返回指定枚举类型的仿函数类型。

那么，怎么使用这个类呢？

`IndexType`是要作为模板参数传进来的，`EnumTypes`……不是现成的么。那么就还缺一个`tuple`类型了。此时可以回顾一下我们在第0步的未雨绸缪了。废话不多说，看码：

```cpp
template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex {
public:
    template <typename IndexType>
    static typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type make_enum;
};

template <typename... EnumTypes>
template <typename IndexType>
typename gen_func_type<IndexType,
    typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
    EnumTypes...>::type
    enum_ex<EnumTypes...>::make_enum
    = nullptr;
```

此时`IndexType`和`EnumTypes`已经准备就绪，然后我们再使用第2步的从`IndexType`和`EnumTypes`找到`enum_item`的`find_enum_item`和第0步的类型萃取，轻轻松松就拿到了生成`enum_ex`的仿函数类型了。我们用它定义一个静态变量`make_enum`。目前我们还没有想好怎么存储这玩意，姑且给它赋值为`nullptr`吧。这里需要注意一点就是`make_enum`这个类型他不是一个常量表达式，不能进行类内初始化，所以必须把他的定义写到类外面，就是最后那一坨。

我们写个小片段测试下编译情况吧：

```cpp
#include "enum_ex.h"

#include <string>

struct Papery {
};
struct Electronic {
};

using Book = enum_ex<
    enum_item<Papery, std::string, int, std::string>,
    enum_item<Electronic, std::string, std::string, std::string, std::string>>;

int main()
{
    Book::make_enum<Papery>("C++变成死相", 30, "Zhang San");
}
```

编译妥妥的。

到这里呢，代码已经变成这样啦（为了防止小班同学跟不上，贴一次完整代码，这绝对不是凑字数哈~~~）：

```cpp
#pragma once

#include <concepts>
#include <functional>
#include <tuple>
#include <type_traits>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = typename std::decay<IndexType>::type;
    using value_type = std::tuple<typename std::decay<PropertyTypes>::type...>;
};

template <typename T>
struct is_enum_item : public std::false_type {
};

template <typename... Types>
struct is_enum_item<enum_item<Types...>> : public std::true_type {
};

template <typename T>
concept EnumTypeConcept = is_enum_item<T>::value;

template <typename IndexType, typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) struct find_enum_item {
    using type = nullptr_t;
};

template <typename IndexType, typename FirstType, typename... EnumTypes>
requires(EnumTypeConcept<typename std::decay<EnumTypes>::type>&&...&& EnumTypeConcept<typename std::decay<FirstType>::type>) struct find_enum_item<IndexType, FirstType, EnumTypes...> {
    using type = typename std::conditional<
        std::is_same<typename std::decay<IndexType>::type, typename FirstType::index_type>::value,
        typename std::decay<FirstType>::type,
        typename find_enum_item<IndexType, EnumTypes...>::type>::type;
};

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex;

template <typename T>
struct is_tuple : std::false_type {
};

template <typename... Types>
struct is_tuple<std::tuple<Types...>> : public std::true_type {
};

template <typename T>
concept TupleTypeConcept = is_tuple<T>::value;

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires(TupleTypeConcept<TupleType>&&...&& EnumTypeConcept<EnumTypes>) struct gen_func_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires(true && ... && EnumTypeConcept<EnumTypes>) struct gen_func_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<enum_ex<EnumTypes...>(TupleTypes...)>;
};

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex {
public:
    template <typename IndexType>
    static typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type make_enum;
};

template <typename... EnumTypes>
template <typename IndexType>
typename gen_func_type<IndexType,
    typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
    EnumTypes...>::type
    enum_ex<EnumTypes...>::make_enum
    = nullptr;
```

突破50行了，很刺激有木有？

### 第4步：把 enum 的属性存起来

这一步就很简单了，就是要把`make_enum`这个函数传进来的参数存起来，后面用的时候还能找到。从上文可知，这个函数接受的参数那可是千奇百怪了，不同类型不同长度都有，那咋办，当然是先填到`tuple`中然后扔到宽容一切的`any`里啦：

```cpp

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex {
private:
private:
    std::any data__;

    template <typename TupleType>
    requires(TupleTypeConcept<TupleType>)
        enum_ex(TupleType tp)
        : data__(tp)
    {
    }

public:
    template <typename IndexType>
    static typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type make_enum;
};

template <typename... EnumTypes>
template <typename IndexType>
typename gen_func_type<IndexType,
    typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
    EnumTypes...>::type
    enum_ex<EnumTypes...>::make_enum
    = typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type([](auto... f) {
          return enum_ex<EnumTypes...>(
              typename find_enum_item<IndexType, EnumTypes...>::type::value_type(f...));
      });
```

这里呢，用到了一丢丢泛型`lambda`，毕竟要真的写类型，才疏学浅的我也不知道怎么写。

上面这一坨代码呢，首先我们把`enum_ex`的构造函数实现了（难得呀），然后`make_enum`直接将传进来的参数包成一个`tuple`丢给`enum_ex`的构造函数，`enum_ex`的构造函数将参数丢给`any`对象存起来。

### 第5步：把属性取出来

上一步把属性存了，现在该把属性取出来了，首先回头看看`Rust`是怎么取的，`Rust`用`match`关键字匹配一个枚举，然后符合某种模式的将属性取出来，然后处理这些属性。`C++`的话，没有`match`……那就写个差不多的吧：

```cpp
template <typename IndexType, typename TupleType, typename... EnumTypes>
requires(TupleTypeConcept<TupleType>&&...&& EnumTypeConcept<EnumTypes>) struct apply_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires(true && ... && EnumTypeConcept<EnumTypes>) struct apply_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<void(TupleTypes...)>;
};

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex {
private:
    std::any data__;

    template <typename TupleType>
    requires(TupleTypeConcept<TupleType>)
        enum_ex(TupleType tp)
        : data__(tp)
    {
    }

public:
    template <typename IndexType>
    static typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type make_enum;

    template <typename IndexType>
    enum_ex<EnumTypes...>& $(typename apply_type<IndexType, typename find_enum_item<IndexType, EnumTypes...>::type::value_type, EnumTypes...>::type f)
    {
        try {
            std::apply(f, std::any_cast<typename find_enum_item<IndexType, EnumTypes...>::type::value_type>(data__));
        } catch (const std::bad_cast&) {
        }
        return *this;
    }
};
```

这段代码的上面那一坨是构造出一个回调函数类型仿函数的类，然后`$`模板函数（没错，`$`就是函数名，毕竟键盘上的符号已经被用光了）以`IndexType`作为模板形参，尝试匹配`any`中存储的类型，如果匹配上，就调用回调函数，展开`tuple`作为函数参数的用法幼儿园小班已经烂熟于心啦，就是`apply`嘛。至于依旧返回`enum_ex`，当然是为了链式调用，把它串起来啦。

写段代码测试一下：

```cpp
#include "enum_ex.h"

#include <iostream>
#include <string>

struct Papery {
};
struct Electronic {
};

using Book = enum_ex<
    enum_item<Papery, std::string, int, std::string>,
    enum_item<Electronic, std::string, std::string, std::string, std::string>>;

int main()
{
    auto e = Book::make_enum<Papery>("C++变成死相", 30, "Zhang San");

    e
        .$<Papery>([](auto name, auto price, auto author) {
            std::cout << "Papery book name: " << name << ", price:" << price << " author:" << author << std::endl;
        })
        .$<Electronic>([](auto name, auto url, auto author, auto filetype) {
            std::cout << "E-book name: " << name << " url:" << url << " author: " << author << " filetype:" << filetype << std::endl;
        });
}
```

成功编译并运行，老哥，稳！

再贴一次当前的实现代码：

```cpp
#pragma once

#include <any>
#include <concepts>
#include <functional>
#include <tuple>
#include <type_traits>
#include <typeinfo>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = typename std::decay<IndexType>::type;
    using value_type = std::tuple<typename std::decay<PropertyTypes>::type...>;
};

template <typename T>
struct is_enum_item : public std::false_type {
};

template <typename... Types>
struct is_enum_item<enum_item<Types...>> : public std::true_type {
};

template <typename T>
concept EnumTypeConcept = is_enum_item<T>::value;

template <typename IndexType, typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) struct find_enum_item {
    using type = nullptr_t;
};

template <typename IndexType, typename FirstType, typename... EnumTypes>
requires(EnumTypeConcept<typename std::decay<EnumTypes>::type>&&...&& EnumTypeConcept<typename std::decay<FirstType>::type>) struct find_enum_item<IndexType, FirstType, EnumTypes...> {
    using type = typename std::conditional<
        std::is_same<typename std::decay<IndexType>::type, typename FirstType::index_type>::value,
        typename std::decay<FirstType>::type,
        typename find_enum_item<IndexType, EnumTypes...>::type>::type;
};

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex;

template <typename T>
struct is_tuple : std::false_type {
};

template <typename... Types>
struct is_tuple<std::tuple<Types...>> : public std::true_type {
};

template <typename T>
concept TupleTypeConcept = is_tuple<T>::value;

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires(TupleTypeConcept<TupleType>&&...&& EnumTypeConcept<EnumTypes>) struct gen_func_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires(true && ... && EnumTypeConcept<EnumTypes>) struct gen_func_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<enum_ex<EnumTypes...>(TupleTypes...)>;
};

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires(TupleTypeConcept<TupleType>&&...&& EnumTypeConcept<EnumTypes>) struct apply_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires(true && ... && EnumTypeConcept<EnumTypes>) struct apply_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<void(TupleTypes...)>;
};

template <typename... EnumTypes>
requires(EnumTypeConcept<EnumTypes>&&... && true) class enum_ex {
private:
    std::any data__;

    template <typename TupleType>
    requires(TupleTypeConcept<TupleType>)
        enum_ex(TupleType tp)
        : data__(tp)
    {
    }

public:
    template <typename IndexType>
    static typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type make_enum;

    template <typename IndexType>
    enum_ex<EnumTypes...>& $(typename apply_type<IndexType, typename find_enum_item<IndexType, EnumTypes...>::type::value_type, EnumTypes...>::type f)
    {
        try {
            std::apply(f, std::any_cast<typename find_enum_item<IndexType, EnumTypes...>::type::value_type>(data__));
        } catch (const std::bad_cast&) {
        }
        return *this;
    }
};

template <typename... EnumTypes>
template <typename IndexType>
typename gen_func_type<IndexType,
    typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
    EnumTypes...>::type
    enum_ex<EnumTypes...>::make_enum
    = typename gen_func_type<IndexType,
        typename find_enum_item<IndexType, EnumTypes...>::type::value_type,
        EnumTypes...>::type([](auto... f) {
          return enum_ex<EnumTypes...>(
              typename find_enum_item<IndexType, EnumTypes...>::type::value_type(f...));
      });
```

### 第6步：一丢丢优化

上面的代码有很多可以优化的地方，包括但不限于：

- 对多个类型限定的`concept`可以写成一个，不用`fold expression`
- 使用`using`和`constexpr`消除`typename xxx::type`和`xxx::value`
- 使用类型自动推导，不显式写出类型

优化后的代码长这样：

```cpp
#pragma once

#include <any>
#include <concepts>
#include <functional>
#include <tuple>
#include <type_traits>
#include <typeinfo>

template <typename IndexType, typename... PropertyTypes>
struct enum_item {
    using index_type = std::decay_t<IndexType>;
    using value_type = std::tuple<std::decay_t<PropertyTypes>...>;
};

template <typename T>
struct is_enum_item : public std::false_type {
};

template <typename... Types>
struct is_enum_item<enum_item<Types...>> : public std::true_type {
};

template <typename T>
constexpr bool is_enum_item_v = is_enum_item<T>::value;

template <typename... Types>
concept EnumTypesConcept = (is_enum_item_v<Types> && ... && true);

template <typename IndexType, typename... EnumTypes>
requires EnumTypesConcept<EnumTypes...> struct find_enum_item {
    using type = nullptr_t;
};

template <typename IndexType, typename FirstType, typename... EnumTypes>
requires EnumTypesConcept<std::decay_t<EnumTypes>..., std::decay_t<FirstType>> struct find_enum_item<IndexType, FirstType, EnumTypes...> {
    using type = std::conditional_t<
        std::is_same<std::decay_t<IndexType>, typename FirstType::index_type>::value,
        std::decay_t<FirstType>,
        typename find_enum_item<IndexType, EnumTypes...>::type>;
};

template <typename IndexType, typename... EnumTypes>
using find_enum_item_t = typename find_enum_item<IndexType, EnumTypes...>::type;

template <typename IndexType, typename... EnumTypes>
using find_enum_item_vt = typename find_enum_item_t<IndexType, EnumTypes...>::value_type;

template <typename... EnumTypes>
requires EnumTypesConcept<EnumTypes...> class enum_ex;

template <typename T>
struct is_tuple : std::false_type {
};

template <typename... Types>
struct is_tuple<std::tuple<Types...>> : public std::true_type {
};

template <typename T>
concept TupleTypeConcept = is_tuple<T>::value;

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires TupleTypeConcept<TupleType>&& EnumTypesConcept<EnumTypes...> struct gen_func_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires EnumTypesConcept<EnumTypes...> struct gen_func_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<enum_ex<EnumTypes...>(TupleTypes...)>;
};

template <typename IndexType, typename TupleType, typename... EnumTypes>
requires TupleTypeConcept<TupleType>&& EnumTypesConcept<EnumTypes...> struct apply_type;

template <typename IndexType, typename... TupleTypes, typename... EnumTypes>
requires EnumTypesConcept<EnumTypes...> struct apply_type<IndexType, std::tuple<TupleTypes...>, EnumTypes...> {
    using type = std::function<void(TupleTypes...)>;
};

template <typename IndexType, typename TupleType, typename... EnumTypes>
using apply_type_t = typename apply_type<IndexType, TupleType, EnumTypes...>::type;

template <typename... EnumTypes>
requires EnumTypesConcept<EnumTypes...> class enum_ex {
private:
    std::any data__;

    template <typename TupleType>
    requires(TupleTypeConcept<TupleType>)
        enum_ex(TupleType tp)
        : data__(tp)
    {
    }

public:
    template <typename IndexType>
    static typename gen_func_type<IndexType, find_enum_item_vt<IndexType, EnumTypes...>,
        EnumTypes...>::type make_enum;

    template <typename IndexType>
    enum_ex<EnumTypes...>& $(apply_type_t<IndexType, find_enum_item_vt<IndexType, EnumTypes...>, EnumTypes...> f)
    {
        try {
            std::apply(f, std::any_cast<find_enum_item_vt<IndexType, EnumTypes...>>(data__));
        } catch (const std::bad_cast&) {
        }
        return *this;
    }
};

template <typename... EnumTypes>
template <typename IndexType>
typename gen_func_type<IndexType,
    find_enum_item_vt<IndexType, EnumTypes...>,
    EnumTypes...>::type
    enum_ex<EnumTypes...>::make_enum
    = [](auto... f) {
          return enum_ex<EnumTypes...>(
              find_enum_item_vt<IndexType, EnumTypes...>(f...));
      };
```

另外附上一坨使用示例：

```cpp
#include "enum_ex.h"

#include <iostream>
#include <string>

struct Papery {
};
struct Electronic {
};

using Book = enum_ex<
    enum_item<Papery, std::string, int, std::string>,
    enum_item<Electronic, std::string, std::string, std::string, std::string>>;

int main()
{
    auto e = Book::make_enum<Papery>("C++变成死相", 30, "Zhang San");
    auto f = Book::make_enum<Electronic>("加瓦入门到精通", "xxxx.xxx.xxx", "Li Si", "pdf");

    auto print_papery = [](auto name, auto price, auto author) {
        std::cout << "Papery book name: " << name << ", price:" << price << " author:" << author << std::endl;
    };

    auto print_electronic = [](auto name, auto url, auto author, auto filetype) {
        std::cout << "E-book name: " << name << " url:" << url << " author: " << author << " filetype:" << filetype << std::endl;
    };

    e
        .$<Papery>(print_papery)
        .$<Electronic>(print_electronic);
    f
        .$<Papery>(print_papery)
        .$<Electronic>(print_electronic);
}
```

和开头的`Rust`代码遥相呼应哦。

后面还可以继续往下做哦，比如可以这样：

```cpp
auto [name, price, author] = e.prop<Papery>();
```

聪明的小盆友一定知道怎么实现啦（小提示：结构化绑定），反正我已经不想写了，现代`C++`的世界就是这么朴实无华，且枯燥~~~

洗洗睡觉~~~
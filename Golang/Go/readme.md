# Go基础

## 命令行

1. go build 编译

`go build`用来编译`go`文件。

一个简单的`hello-world`程序

```go
// main.go
package main

import "fmt"

func main(){
	fmt.Println("Hello World")
}
```

使用`go build` 编译：

```bash
go build
```

或者：

```bash
go build main.go
```

也可以指定文件名

```bash
go build -o hello main.go 
```

这样就会在当前目录生成可执行文件

2. go run 脚本执行

继续上一个`hello world`程序。

如果我们在写完代码后，不想编译，而是想里面看到结果，我们可以使用`go run`命令直接以脚本的方式执行。

```bash
go run main.go
```

3. go fmt 格式化代码

`fmt`是个有用的工具，可以帮你把杂乱的代码格式化为正常格式。将前面的`hello world`代码格式打乱：

```go
// main.go
      		package main

		import "fmt"



			func main(){
fmt.Println("Hello World")
}
```

然后运行

```bash
go fmt main.go
```

瞬间就变得整整齐齐。

4. go doc 查看文档

如果我们要查看`fmt.Println`的文档，只需要：

```bash
go doc fmt.Println
```

初次之外，`go`还提供`web`服务器查看文档方式：

```bash
godoc -http :8888
```

在浏览器打开[http://localhost:8888](http://localhost:8888)就能访问文档了。

5. go get 下载包

`go`的模块是以包的形式提供的，网上有很多优秀的包，可以通过`go get`命令下载安装，假如我们要使用`github.com/astaxie/beego`这个包:

```bash
go get github.com/astaxie/beego
```

6. go install 安装

要说`go install`，免不了说`GOPATH`。

`GOPATH`这个环境变量其实就是`Golang`的一个开发环境，里面包括三个目录: `bin`、`pkg`、`src`

`bin`目录主要存放可执行文件; `pkg`目录存放编译好的库文件, 主要是`*.a`文件; `src`目录下主要存放`go`的源文件

在`src`编写代码，执行`go install`后，会将编译后的文件放在`bin`目录，依赖包放在`pkg`，

## 变量

Golang的变量声明方式有以下几种：

1. var

```go
var a int
```
可以同时初始化：

```go
var a int = 5
```

还可以批量定义：

```go
var (
    a int
    b int = 10
) 
```

可以同时定义多个变量，并且支持同时赋值：

```go
var a,b int = 5,6
```

2. :=

通过`:=`的方式，编译器会自动推导类型

```go
a := 5
b := 6
```

或者：

```go
a, b := 5, 6
```

## 常量

常量定义与变量定义类似，只是将`var`换成`const`

## 类型

1. 布尔型 bool
2. 整数
    * int8
    * int16
    * int32
    * int64
    * uint8
    * uint16
    * uint32
    * uint64
    * int(大小与平台相关)
    * uint(大小与平台相关)
    * byte
    * rune(与int32类似)
    * uintptr
    
3. 浮点数
    * float32
    * float64
    * complex64
    * complex128
    
4. 字符串类型
    * string
    
5. 其他
    * 指针
    * 结构体
    * 数组
    * 切片
    * 函数
    * Channel
    * 接口
    * map

## 函数 

函数用`func`声明，格式为

```go
func FuncName(param1 type1, param2 type2) return_type {
	// func body
}
```

左花括号必须与`func`在同一行，如：

```go
package main

import "fmt"

func Add(a,b int) int{
	return a+b
}


func main() {
	fmt.Println(5, "+", 10, "=", Add(5, 10))
}
```

## 包

main.go的源码:

```go
package main

import "fmt"
import "foo"

func main() {
	bar.Abc()
	fmt.Print("This is main\n")
}
```

foo/test.go的源码:
```go

package bar

import "fmt"

func Abc() {
	fmt.Print("This is test print\n")
}
```
上面的代码是顺利通过编译的。我们可以总结以下几点：

(1)import语句使用的是文件夹的名称

上面例子中的import后面的参数对应的就是文件夹foo

(2)文件夹的名称和package的名称不一定相同

上面的例子中，文件夹是foo，而package名称是bar。

(3)调用自定义包使用package名称.函数名的方式

例如上面使用的bar.Abc()。

(4)自定义包的调用和文件名没有关系

例如上面的test.go文件，如果改成test_abc.go，程序也能正常编译。编译系统会自动查找foo文件夹下的所有文件，在其中寻找package bar，然后选择Abc函数。

## 数组

## 切片
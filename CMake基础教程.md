# CMake基础教程

**skyfire**

*2018/01/26*

环境使用MinGW64，GCC版本：7.2，CMake版本3.10.2

文中使用的目录均为项目根目录的相对目录。

## step1.一个基本的单文件c++应用程序

源文件：`src/src1.cpp`:

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout<<"hello world"<<endl;
    return 0;
}
```

编写`CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.9)
project(step1)
add_executable(step1 src/src1.cpp)
```

创建`build`目录，在`build`目录下执行：

```bat
cmake .. -G "MinGW Makefiles"
```

然后使用`mingw32-make`编译得到结果，在`build`目录下生成`step1.exe`

## step2.控制版本号(配置)

源文件`src/src2.cpp`:

```cpp
#include <iostream>
#include <version.h>
using namespace std;

int main()
{
    cout<<"VERSION:"<<APP_VERSION<<endl;
    return 0;
}
```

`version.h`由CMake生成。编写配置文件`src/version.h.in`

```cpp
#ifndef VERSION_H_
#define VERSION_H_

#define APP_VERSION @APP_VERSION@

#endif
```

编写`CMakelist.txt`：

```cmake
cmake_minimum_required(VERSION 3.9)
project(step2)

set(APP_VERSION "\"V1.0.0.0\"")

configure_file(
    ${PROJECT_SOURCE_DIR}/src/version.h.in
    ${PROJECT_BINARY_DIR}/version.h
)

include_directories(${PROJECT_BINARY_DIR})

add_executable(step2 src/src2.cpp)
```

CMake设置变量APP_VERSION为`"V1.0.0.0"`，然后设置配置文件为`src/version.h.in`，输出文件为`build/version.h` (`build`为编译目录)，随后增加包含目录`build`。

然后使用`cmake .. -G "MinGW Makefiles"`使用生成`Makefile`，同时会生成`build/version.h`。

生成后使用`mingw32-make`进行构建，在`build`下生成`step2.exe`

运行`step2.exe`，显示输出：

```txt
V1.0.0.0
```

## step3.生成动态库

编写动态库头文件`include/src3.h`，（如果是MSVC，需要使用`__declspec(dllexport)`导出符号）

```cpp
#ifndef _SRC3_H_
#define _SRC3_H_
#ifdef __cplusplus
extern "C"
{
#endif

int add(int a,int b);

#ifdef __cplusplus
}
#endif
#endif
```

编写动态库源文件`src/src3.cpp`:

```cpp
#include "src3.h"

int add(int a, int b)
{
    return a+b;
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step3)

include_directories(${PROJECT_SOURCE_DIR}/include)

add_library(step3 SHARED src/src3.cpp)
```

动态库的头文件位于`include`文件夹下，所以使用`include_directories`添加头文件目录。

然后使用`add_library`生成库文件，`SHARED`表明这是一个动态库。

使用`cmake`和`make`生成后，在`build`目录下生成`libstep3.dll`和`libstep3.dll.a`。

## step4.生成静态库

生成静态库和生成动态库的方法基本一致，只是在`add_library`中将`SHARED`替换为`STATIC`

静态库头文件`include/src4.h`:

```cpp
#ifndef _SRC4_H_
#define _SRC4_H_
#ifdef __cplusplus
extern "C"
{
#endif

int add(int a,int b);

#ifdef __cplusplus
}
#endif
#endif
```

静态库源文件`src/src4.cpp`:

```cpp
#include "src4.h"

int add(int a, int b)
{
    return a+b;
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step4)

include_directories(${PROJECT_SOURCE_DIR}/include)

add_library(step4 STATIC src/src4.cpp)
```

注意此处将`SHARED`替换为了`STATIC`

然后`cmake`、`make`，在`build`目录下生成`libstep4.a`。

## step5.使用动/静态库

动态库与静态库的使用方法相同。

这一步使用`step4`中生成的静态库`libstep4.a`

将`libstep4.a`复制到`lib`目录，将库的头文件`src4.h`复制到`include`目录。

编写源文件src5.cpp`:

```cpp
#include <iostream>
#include "src4.h"

using namespace std;

int main()
{
    cout << add(12 ,13) << endl;
    return 0;
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step5)

include_directories(${PROJECT_SOURCE_DIR}/include)
link_directories(${PROJECT_SOURCE_DIR}/lib)

add_executable(step5 src/src5.cpp)

target_link_libraries(step5 step4)
```

使用`include_directories`添加包含目录，使用`link_directories`添加库目录。

使用`target_link_libraries`设置生成目标需要链接的库。

执行`cmake`和`make`后，生成`step5.exe`，运行`step5.exe`:

```txt
25
```

## step6.子目录

一个解决方案通常会包含多个项目，这次我们写一个稍微复杂的`demo`。

建立工程目录结构如下：

```txt
├─build
├─proj1
│  ├─include
│  └─src
├─proj2
│  ├─include
│  └─src
└─proj3
    ├─include
    └─src
```

其中`proj3`为主程序，`proj1`为动态库程序，`proj2`为静态库程序。

编写`proj1`的头文件、源文件、cmake文件如下：

`proj1/inlcude/add.h`:

```cpp
#ifndef _ADD_H_
#define _ADD_H_
#ifdef __cplusplus
extern "C"
{
#endif

int add(int a,int b);

#ifdef __cplusplus
}
#endif
#endif
```

`proj1/src/add.cpp`:

```cpp
#include "add.h"

int add(int a,int b)
{
    return a+b;
}
```

`proj1/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(proj1)

include_directories(${PROJECT_SOURCE_DIR}/include)

add_library(proj1 SHARED src/add.cpp)
```

编写`proj2`的头文件、源文件、cmake文件如下：

`proj2/inlcude/mul.h`:

```cpp
#ifndef _MUL_H_
#define _MUL_H_
#ifdef __cplusplus
extern "C"
{
#endif

int mul(int a,int b);

#ifdef __cplusplus
}
#endif
#endif
```

`proj2/src/mul.cpp`:

```cpp
#include "mul.h"

int mul(int a,int b)
{
    return a*b;
}
```

`proj2/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(proj2)

include_directories(${PROJECT_SOURCE_DIR}/include)

add_library(proj2 STATIC src/mul.cpp)
```

编写`proj3`的源文件、cmake文件如下：（`proj3`暂时没有头文件）

`proj3/src/app.cpp`:

```cpp
#include <iostream>
#include "add.h"
#include "mul.h"

using namespace std;

int main()
{
    cout << add(5,6) <<endl;
    cout << mul(5,6) <<endl;
    return 0;
}
```

`proj3/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(proj3)

include_directories(
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/../proj1/include
    ${PROJECT_SOURCE_DIR}/../proj2/include
    )

link_directories(
    ${PROJECT_BINARY_DIR}/../proj1
    ${PROJECT_BINARY_DIR}/../proj2
    )

add_executable(proj3 src/app.cpp)

target_link_libraries(proj3 proj1 proj2)
```

编写整个解决方案的`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step6)

add_subdirectory(proj1)
add_subdirectory(proj2)
add_subdirectory(proj3)
```

执行`cmake`和`make`，生成`proj1/libproj1.dll`、`proj2/libproj2.a`、`proj3/proj3.exe`。将`proj1/libproj1.dll`复制到`proj3/`目录下，运行`proj3/proj3.exe`（动态库在运行时需要）:

```txt
11
30
```

## step7.选项

有时在构建过程中，需要有条件编译。

编写例子如下：

源文件`src/src7.cpp`：

```cpp
#include <iostream>
#include <config.h>

using namespace std;

int main()
{
    #ifdef APP_CONFIG
    cout<<"Use Config"<<endl;
    #else
    cout<<"Don't Use Config" << endl;
    #endif
}
```

配置文件`src/config.h.in`

```cpp
#ifndef _CONFOG_H_
#define _CONFOG_H_

#cmakedefine APP_CONFIG

#endif
```

`cmake`文件`CMakefiles.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step7)

option(
    APP_CONFIG
    "Use config1?"
    ON
)

configure_file(
    ${PROJECT_SOURCE_DIR}/src/config.h.in
    ${PROJECT_BINARY_DIR}/config.h
)

include_directories(${PROJECT_BINARY_DIR})

add_executable(step7 src/src7.cpp)
```

使用`option`可以定义一个选项，例子中默认值为`ON`，在`CMake-GUI`程序中，可以配置这个选项。

`cmake`并`make`后，生成`build/step7.exe`,运行结果

```txt
Use Config
```

（当默认值设为`OFF`时，输出为`Don't Use Config`）

## step8.安装

此次编写一个动态库，并安装。

动态库头文件`include/src8.h`:

```cpp
#ifndef _SRC8_H_
#define _SRC8_H_
#ifdef __cplusplus
extern "C"
{
#endif

int add(int a,int b);

#ifdef __cplusplus
}
#endif
#endif
```

动态库源文件`src/src8.cpp`:

```cpp
#include "src8.h"

int add(int a, int b)
{
    return a+b;
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step8)

include_directories(${PROJECT_SOURCE_DIR}/include)

set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR}/install)

add_library(step8 SHARED src/src8.cpp)

install(TARGETS step8 DESTINATION lib)
install(FILES ${PROJECT_SOURCE_DIR}/include/src8.h DESTINATION include)
```

设置`CMAKE_INSTALL_PREFIX`变量的值可以指定安装目录。使用`install`可以添加安装功能。

其中`TARGETS`选项表示安装一个工程（动态库安装`.dll`,`.dll.a`，静态库安装`.a`，应用程序安装`.exe`），`FILES`选项安装文件（通常是工程生成文件之外的文件，如本例中的头文件）。

执行`cmake`和`make`后，执行`make install`，查看`install`下文件：

```txt
├─include
│      src8.h
│
└─lib
        libstep8.dll
        libstep8.dll.a
```

## step9.测试

编写一个应用程序：

源文件`src/srcp.cpp`:

```cpp
#include <iostream>

using namespace std;

int main(int argc ,char ** argv)
{
    if (argc == 1)
    {
        cout<<"No params"<<endl;
    }
    else
    {
        cout << argv[1] << " passed" <<endl;
    }
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step9)

add_executable(step9 src/src9.cpp)

include(CTest)

macro (do_test test_name arg result)
  add_test (test${test_name} step9 ${arg})
  set_tests_properties (test${test_name} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

do_test(1 "hello" "hello passed")
do_test(2 "" "No params")
do_test(3 "hello world" "hello world passed")
```

使用测试功能需要`include(CTest)`，然后定义了一个宏`do_test`，这个宏用于生成测试模板。

接下来编写了三个测试用例。

执行`cmake`和`make`后，执行`make test`，会执行测试用例并输出测试结果。

## step10.函数判别

这个例子中，我们实现一个程序，判断、`log`函数和`alloca`函数的存在性进行条件编译。

源文件`src/src10.cpp`:

```cpp
#include <iostream>
#include <cmath>
#include "config.h"

using namespace std;

int main()
{
    #ifdef HAS_LOG
    cout<<"has log, log(5)=" << log(5) <<endl;
    #else
    cout<<"don't have log"<<endl;
    #endif
    #ifdef HAS_ALLOC_A
    cout<<"has alloca" <<endl;
    #else
    cout<<"don't have alloca"<<endl;
    #endif
}
```

配置文件`src/config.h.in`:

```cpp
#cmakedefine HAS_LOG
#cmakedefine HAS_ALLOC_A
```

`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step10)

include(CheckFunctionExists)

check_function_exists(log HAS_LOG)
check_function_exists(alloca HAS_ALLOC_A)

configure_file(
    ${PROJECT_SOURCE_DIR}/src/config.h.in
    ${PROJECT_BINARY_DIR}/config.h
)

include_directories(${PROJECT_BINARY_DIR})

add_executable(step10 src/src10.cpp)
```

使用`include(CheckFunctionExists)`引入函数检测模块，使用`check_function_exists`检测函数是否存在，并定义相应的变量。如：如果加测到`log`函数存在，就定义`HAS_LOG`变量。然后根据定义的变量影响`config.h.in`配置文件生成的配置头文件。

进行`cmake`和`make`过程后，生成`build/step10.exe`，运行得到：

```txt
has log, log(5)=1.60944
don't have alloca
```

(不同环境运行结果不同)

其他的一些检测，可以引入响应的`Check`模块进行，比如`include(CheckCXXSymbolExists)`等。

## step11.附加命令

此例子在编译过程中生成新的文件。

源文件`src/make_my.cpp`

```cpp
#include <iostream>
#include <fstream>

using namespace std;

int main(int argc, char ** argv)
{
    if(argc != 2)
    {
        cout<<"params count error"<<endl;
        return -1;
    }
    else
    {
        ofstream of(argv[1]);
        of<<"#define MY_DEFINE"<<endl;
        of.close();
        return 0;
    }
}
```

此文件根据命令函参数，往指定文件中写入一个宏定义

源文件`src/src11.cpp`:

```cpp
#include <iostream>
#include "my.h"
#include "my2.h"

using namespace std;

int main()
{
    cout<<"hello world"<<endl;
    #ifdef MY_DEFINE
    cout<<"MY_DEFINE defined"<<endl;
    #endif
    #ifdef MY_DEFINE2
    cout<<"MY_DEFINE2 defined"<<endl;
    #endif
    return 0;
}
```

编写`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.9)
project(step11)

add_executable(make_my src/make_my.cpp)

add_custom_command(
    OUTPUT ${PROJECT_BINARY_DIR}/my.h
    COMMAND make_my ${PROJECT_BINARY_DIR}/my.h
    DEPENDS make_my
)

add_custom_command(
    OUTPUT ${PROJECT_BINARY_DIR}/my2.h
    COMMAND echo \#define MY_DEFINE2 > ${PROJECT_BINARY_DIR}/my2.h
)

include_directories(${PROJECT_BINARY_DIR})

add_executable(step11 
    src/src11.cpp 
    ${PROJECT_BINARY_DIR}/my.h 
    ${PROJECT_BINARY_DIR}/my2.h
)
```

该文件中，添加了两个自定义命令，第一个调用`make_my`命令，此命令要使用的`make_my`程序由`make_my.cpp`编译而成。生成`build/my.h`文件。第二个命令直接用`echo`生成`build/my2.h`文件。

然后由生成的两个头文件和`src/src11.cpp`生成`build/step11.exe`。（生成的依赖一定要包含动态生成的头文件）

执行`cmake`和`make`，生成`build/step11.exe`，运行结果：

```txt
hello world
MY_DEFINE defined
MY_DEFINE2 defined
```

## step12.制作安装包

接下来我们来用`cmake`制作一个安装包：

源文件`src/src12.cpp`:

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout<<"hello world"<<endl;
    return 0;
}
```

在`src`下放置一个`License.txt`文件（不是必需的）

`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.7)
project(step12)
add_executable(step12 src/src12.cpp)

set(CMAKE_INSTALL_PREFIX /home/skyfire)

install(
    TARGETS step12 DESTINATION bin
)

include(InstallRequiredSystemLibraries)

set (CPACK_RESOURCE_FILE_LICENSE  
     ${PROJECT_SOURCE_DIR}/src/License.txt)
set (CPACK_PACKAGE_VERSION_MAJOR "1")
set (CPACK_PACKAGE_VERSION_MINOR "2")

include (CPack)
```

该脚本先设置了安装信息。然后使用`include(InstallRequiredSystemLibraries)`打包需要的系统运行库，然后设置`CPack`的相关参数，例子中有许可信息，主次版本号。

执行`cmake`和`make`，然后`make package`，在linux系统下，生成如下文件：

```txt
CMakeCache.txt
CMakeFiles
cmake_install.cmake
CPackConfig.cmake
_CPack_Packages
CPackSourceConfig.cmake
install_manifest.txt
Makefile
step12
step12-1.2.1-Linux.sh
step12-1.2.1-Linux.tar.gz
step12-1.2.1-Linux.tar.Z
```

在Windows系统下生成(MinGW)：（Windows需要安装NSIS）

```txt
CMakeCache.txt
CMakeFiles
cmake_install.cmake
CPackConfig.cmake
CPackSourceConfig.cmake
install_manifest.txt
Makefile
step12-1.2.1-win32.exe
step12.exe
_CPack_Packages
```

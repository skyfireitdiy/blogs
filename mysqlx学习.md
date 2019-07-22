# mysqlx学习

资料：[https://dev.mysql.com/doc/dev/connector-cpp/8.0/devapi_ref.html](https://dev.mysql.com/doc/dev/connector-cpp/8.0/devapi_ref.html)

## 注意事项

1. 使用高版本的gcc编译的时候，需要定义宏`-D_GLIBCXX_USE_CXX11_ABI=0`,否则会出现链接错误。

2. 不要同时`using namespace std;`和`using namespace mysqlx`，因为`string`会发生冲突。

3. 本文测试的环境为：
   - os：`Linux skyfire-pc 4.19.59-1-MANJARO #1 SMP PREEMPT Mon Jul 15 18:23:58 UTC 2019 x86_64 GNU/Linux`
   - gcc：9.1.0

   - mysqlx 位置：`/home/skyfire/Desktop/mysql-connector-c++-8.0.16-linux-glibc2.12-x86-64bit`

   - mysql 版本：8.0.16 
   - mysql 用户名：root
   - mysql 密码：root

## 连接数据库

CMakeLists.txt 如下：

```cmake
cmake_minimum_required(VERSION 3.14)
project(testMysql)

set(CMAKE_CXX_STANDARD 14)

link_directories("/home/skyfire/Desktop/mysql-connector-c++-8.0.16-linux-glibc2.12-x86-64bit/lib64")
include_directories("/home/skyfire/Desktop/mysql-connector-c++-8.0.16-linux-glibc2.12-x86-64bit/include")

add_executable(testMysql main.cpp)

target_link_libraries(testMysql mysqlcppconn8)

add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)

```

（无特殊说明，以下会继续使用此配置）

代码如下（为了减少代码行数，已经去掉了异常处理）：

```cpp
#include <mysqlx/xdevapi.h>


int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);
    std::cout<<"connected"<<std::endl;
}
```

编译运行。如果连接失败，会抛出异常。

## 创建/获取 Schema

代码如下：

```cpp
#include <mysqlx/xdevapi.h>


int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);

    auto schema = mysql_session.createSchema("test", true);
    // auto schema = mysql_session.getSchema("test", true);
}
```

代码创建一个Schema， 如果已存在，就使用已有的Schema。

如果第二个参数为 false ，在Schema已存在时，会抛出异常。

下面注释的部分应用于确定已存在Schema的情况，第二个参数意义为是否立即检查Schema的存在性。

## 表存储（SQL）

在数据库创建一个表：

```SQL
CREATE TABLE test.student(
    name VARCHAR(32),
    age INT,
    address VARCHAR(64)
);
```

### 获取表

```cpp
int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);

    auto schema = mysql_session.createSchema("test", true);
    // auto schema = mysql_session.getSchema("test", true);

    auto table = schema.getTable("student", true);
}
```

`getTable` 可以获取一张表，第二个参数代表是否检测表的存在性。

### CRUD 操作

```cpp
#include <mysqlx/xdevapi.h>
#include <iostream>


int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);

    auto schema = mysql_session.createSchema("test", true);
    // auto schema = mysql_session.getSchema("test", true);

    auto table = schema.getTable("student", true);

    // remove all
    table.remove().execute();

    auto show_all_data = [&]{
        auto result = table.select().execute();
        auto data = result.fetchAll();
        for (auto d: data){
            std::cout<<"name:"<<d.get(0)<<" age:"<<d.get(1)<<" address:"<<d.get(2)<<std::endl;
        }
        std::cout<<"================="<<std::endl;
    };

    // insert into test.student values('skyfire', 26, "Xi'an");
    table.insert().values("skyfire", 26, "Xi'an").execute();
    // insert into test.student(age, address, name) values(28, 'beijing', 'zhangsan');
    table.insert("age", "address", "name").values(28, "beijing", "zhangsan").execute();

    show_all_data();

    // update test.student set name='lisi' where name='zhangsan';
    table.update().set("name", "lisi").where("name = 'zhangsan'").execute();

    show_all_data();

    // delete from test.student where name = 'skyfire';
    table.remove().where("name = 'skyfire'").execute();

    show_all_data();
}
```

## 文档存储（NoSql）

### 创建/获取集合

```cpp
#include <mysqlx/xdevapi.h>

int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);

    auto schema = mysql_session.createSchema("test", true);
    // auto schema = mysql_session.getSchema("test", true);

    auto coll = schema.createCollection("student", true);
    // auto coll = schema.getCollection("student", true);
}
```

在Schema上创建一个集合，getCollection 函数与 createCollection 函数的区别与Schema相关函数类似，不再赘述。

### CRUD 操作

mysqlx 对文档的操作很友好，大致看一下就明白含义了。

```cpp
#include <mysqlx/xdevapi.h>
#include <iostream>


int main() {
    std::string conf = "mysqlx://root:root@127.0.0.1:33060";
    mysqlx::Session mysql_session(conf);

    auto schema = mysql_session.createSchema("test", true);
    // auto schema = mysql_session.getSchema("test", true);

    auto coll = schema.createCollection("student", true);
    // auto coll = schema.getCollection("student", true);

    auto show_all_data = [&]{
        auto result = coll.find("true").execute();
        auto res_list = result.fetchAll();
        for(auto p:res_list)
        {
            std::cout<<p<<std::endl;
        }
        std::cout<<"=============="<<std::endl;
    };
    // delele all data from collection "student"
    coll.remove("true").execute();

    // add two document to "student" with different key
    coll.add(R"({"name":"skyfire", "age": 26, "address":"Xi'an"})").execute();
    coll.add(R"({"name":"zhangsan", "phone": "123465789", "tail":180})").execute();

    show_all_data();

    // update document add property 'sex' = 'female' and set 'phone' = '000000000'
    coll.modify("name = 'zhangsan'").set("phone", "000000000").set("sex", "female").execute();

    show_all_data();

    // delete document which sex is not null
    coll.remove("sex is not null").execute();

    show_all_data();
}
```

输出：

```text
{"_id": "00005d301af80000000000000013", "age": 26, "name": "skyfire", "address": "Xi'an"}
{"_id": "00005d301af80000000000000014", "name": "zhangsan", "tail": 180, "phone": "123465789"}
==============
{"_id": "00005d301af80000000000000013", "age": 26, "name": "skyfire", "address": "Xi'an"}
{"_id": "00005d301af80000000000000014", "sex": "female", "name": "zhangsan", "tail": 180, "phone": "000000000"}
==============
{"_id": "00005d301af80000000000000013", "age": 26, "name": "skyfire", "address": "Xi'an"}
==============
```


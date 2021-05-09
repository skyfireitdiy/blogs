#  gtest 入门
  
  
##  简介
  
  
Gtest是Google公司发布的一款非常优秀的开源C/C++单元测试框架，已被应用于多个开源项目及Google内部项目中，知名的例子包括ChromeWeb浏览器、LLVM编译器架构、ProtocolBuffers数据交换格式及工具等。在我们开发规范的代码时候，要想办法构造简单的测试用例进行调试，因此针对gtest中的三种事件机制进行简单的分析。
  
##  使用
  
  
下面仅列举一些比较常用的测试功能，需要更详细的介绍可参考博客：[玩转Google开源C++单元测试框架Google Test系列](https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html )
  
###  断言
  
  
断言用于直接判断表达式执行是否符合预期的结果，如：返回值是否符合预期、是否抛出预期的异常等。
  
下面仅演示一下基本结构。
  
头文件：rect.h
```h
#ifndef _CALC_H_
#define _CALC_H_
#ifdef __cplusplus
extern "C" {
#endif
  
    int RectArea(int x1, int y1, int x2, int y2);
  
#ifdef __cplusplus
}
#endif
#endif
```  
源文件：rect.c
```c
#include "rect.h"
  
int RectArea(int x1, int y1, int x2, int y2){
    return (y2 - y1) * (x2 - x1);
}
```  
测试用例：test.cpp
```cpp
#include "rect.h"
#include <gtest/gtest.h>
  
TEST(TestRect, TestRectArea) { 
    EXPECT_EQ(RectArea(0, 0, 5, 5), 25); 
}
```  
测试入口：main.cpp
```cpp
#include <gtest/gtest.h>
  
int main(int argc, char** argv){
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```  
  
```{code_chunk_offset=0,
运行结果：
```

```
build ok!
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from TestRect
[ RUN      ] TestRect.TestRectArea
[       OK ] TestRect.TestRectArea (0 ms)
[----------] 1 test from TestRect (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.
```

  
断言的种类有以下几种：
  
####  布尔值检查
  
  
| Fatal assertion          | Nonfatal assertion       | Verifies           |
| :----------------------- | :----------------------- | :----------------- |
| ASSERT_TRUE(condition);  | EXPECT_TRUE(condition);  | condition is true  |
| ASSERT_FALSE(condition); | EXPECT_FALSE(condition); | condition is false |
  
####  数值型数据检查
  
  
  
| Fatal assertion              | Nonfatal assertion           | Verifies           |
| :--------------------------- | :--------------------------- | :----------------- |
| ASSERT_EQ(expected, actual); | EXPECT_EQ(expected, actual); | expected == actual |
| ASSERT_NE(val1, val2);       | EXPECT_NE(val1, val2);       | val1 != val2       |
| ASSERT_LT(val1, val2);       | EXPECT_LT(val1, val2);       | val1 < val2        |
| ASSERT_LE(val1, val2);       | EXPECT_LE(val1, val2);       | val1 <= val2       |
| ASSERT_GT(val1, val2);       | EXPECT_GT(val1, val2);       | val1 > val2        |
| ASSERT_GE(val1, val2);       | EXPECT_GE(val1, val2);       | val1 >= val2       |
  
####  字符串检查
  
  
| Fatal assertion                             | Nonfatal assertion                          | Verifies                                                |
| :------------------------------------------ | :------------------------------------------ | :------------------------------------------------------ |
| ASSERT_STREQ(expected_str, actual_str);     | EXPECT_STREQ(expected_str, actual_str);     | the two C strings have the same content                 |
| ASSERT_STRNE(str1, str2);                   | EXPECT_STRNE(str1, str2);                   | the two C strings have different content                |
| ASSERT_STRCASEEQ(expected_str, actual_str); | EXPECT_STRCASEEQ(expected_str, actual_str); | the two C strings have the same content, ignoring case  |
| ASSERT_STRCASENE(str1, str2);               | EXPECT_STRCASENE(str1, str2);               | the two C strings have different content, ignoring case |
  
`STREQ`和`STRNE`同时支持`char*`和`wchar_t*`类型的，`STRCASEEQ`和`STRCASENE`却只接收`char*`
  
####  显式成功或者失败
  
  
成功直接调用 `SUCCEED()`
  
返回失败：
  
`FAIL()`直接返回失败，不再向下执行。
  
`ADD_FAILURE() << "msg"`，失败并显示信息，但是仍向下执行。
  
####  异常检查
  
  
| Fatal assertion                          | Nonfatal assertion                       | Verifies                                        |
| :--------------------------------------- | :--------------------------------------- | :---------------------------------------------- |
| ASSERT_THROW(statement, exception_type); | EXPECT_THROW(statement, exception_type); | statement throws an exception of the given type |
| ASSERT_ANY_THROW(statement);             | EXPECT_ANY_THROW(statement);             | statement throws an exception of any type       |
| ASSERT_NO_THROW(statement);              | EXPECT_NO_THROW(statement);              | statement doesn't throw any exception           |
  
####  浮点数检查
  
  
| Fatal assertion                     | Nonfatal assertion                  | Verifies                                                                     |
| :---------------------------------- | :---------------------------------- | :--------------------------------------------------------------------------- |
| ASSERT_FLOAT_EQ(expected, actual);  | EXPECT_FLOAT_EQ(expected, actual);  | the two float values are almost equal                                        |
| ASSERT_DOUBLE_EQ(expected, actual); | EXPECT_DOUBLE_EQ(expected, actual); | the two double values are almost equal                                       |
| ASSERT_NEAR(val1, val2, abs_error); | EXPECT_NEAR(val1, val2, abs_error); | the difference between val1 and val2 doesn't exceed the given absolute error |
  
  
断言不止上面列出的这么多，这里只列举一下可能会比较常用的，还有一些基本不会用到就没有列举了。
  
  
###  参数化
  
  
参数化应对于对同一个函数传递各种不同参数的测试情况。
  
举例说明：
  
测试用例：test.cpp
```cpp
#include "rect.h"
#include <gtest/gtest.h>
  
struct TestCase {
    int pX1, pX2, pY1, pY2;
    int expectArea;
};
  
bool RunCase(const TestCase& cs){
    return RectArea(cs.pX1, cs.pX2, cs.pY1, cs.pY2) == cs.expectArea;
}
  
  
/**
 * 1. 定义一个类，继承自 testing::TestWithParam<TestCase>
 * 其中 TestCase 就是想要批量传入的参数类型
 * 这个宏会定义一个测试类 RectTest ，下面的步骤会使用这个类
 */
  
class RectTest : public testing::TestWithParam<TestCase> {
};
  
/**
 * 2. 使用 TEST_P 宏告诉测试框架怎么使用每个数据
 * 第一个参数是上面生成的类名 RectTest 
 * 使用 GetParam 函数获取每组测试数据
 */
  
TEST_P(RectTest, RectAreaTest)
{
    TestCase cs = GetParam();
    EXPECT_TRUE(RunCase(cs));
}
  
  
/**
 * 3. 使用INSTANTIATE_TEST_CASE_P 宏批量传入我们的测试数据
 * testing::Values 里面可以放多组测试数据
 */
  
INSTANTIATE_TEST_CASE_P(
    ValidData, 
    RectTest, 
    testing::Values(
        TestCase{0,0,0,0,0},
        TestCase{0,0,5,5,25},
        TestCase{3,4,5,6,4},
        TestCase{-2,-4,2,4,32}
    )
);
  
  
```  
  
```{code_chunk_offset=1,
运行结果：
```

```
build ok!
[==========] Running 4 tests from 1 test case.
[----------] Global test environment set-up.
[----------] 4 tests from ValidData/RectTest
[ RUN      ] ValidData/RectTest.RectAreaTest/0
[       OK ] ValidData/RectTest.RectAreaTest/0 (0 ms)
[ RUN      ] ValidData/RectTest.RectAreaTest/1
[       OK ] ValidData/RectTest.RectAreaTest/1 (0 ms)
[ RUN      ] ValidData/RectTest.RectAreaTest/2
[       OK ] ValidData/RectTest.RectAreaTest/2 (0 ms)
[ RUN      ] ValidData/RectTest.RectAreaTest/3
[       OK ] ValidData/RectTest.RectAreaTest/3 (0 ms)
[----------] 4 tests from ValidData/RectTest (0 ms total)

[----------] Global test environment tear-down
[==========] 4 tests from 1 test case ran. (1 ms total)
[  PASSED  ] 4 tests.
```

  
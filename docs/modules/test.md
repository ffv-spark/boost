# Boost.Test - 单元测试框架

## 概述

Boost.Test 提供强大的单元测试框架，支持测试用例组织、断言、fixture 和测试报告。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#define BOOST_TEST_MODULE MyTest
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_CASE(test_addition) {
    BOOST_TEST(2 + 2 == 4);
    BOOST_TEST(3 + 3 == 6);
}

BOOST_AUTO_TEST_CASE(test_multiplication) {
    BOOST_TEST(2 * 3 == 6);
    BOOST_TEST(4 * 5 == 20);
}
```

**编译**: `g++ -std=c++14 test.cpp -o test && ./test`

---

## 基本断言

```cpp
#define BOOST_TEST_MODULE AssertionTest
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_CASE(test_assertions) {
    // 基本断言
    BOOST_TEST(true);
    BOOST_CHECK(2 + 2 == 4);
    
    // 相等断言
    BOOST_CHECK_EQUAL(10, 5 + 5);
    
    // 不等断言
    BOOST_CHECK_NE(10, 20);
    
    // 小于/大于
    BOOST_CHECK_LT(5, 10);
    BOOST_CHECK_GT(10, 5);
    BOOST_CHECK_LE(5, 5);
    BOOST_CHECK_GE(10, 10);
    
    // 浮点数比较
    BOOST_CHECK_CLOSE(3.14159, 3.14, 1.0);  // 1% 容差
}
```

---

## 测试套件

```cpp
#define BOOST_TEST_MODULE TestSuite
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_SUITE(MathTests)

BOOST_AUTO_TEST_CASE(test_addition) {
    BOOST_CHECK_EQUAL(2 + 2, 4);
}

BOOST_AUTO_TEST_CASE(test_subtraction) {
    BOOST_CHECK_EQUAL(5 - 3, 2);
}

BOOST_AUTO_TEST_SUITE_END()

BOOST_AUTO_TEST_SUITE(StringTests)

BOOST_AUTO_TEST_CASE(test_concatenation) {
    std::string s = "Hello" + std::string(" World");
    BOOST_CHECK_EQUAL(s, "Hello World");
}

BOOST_AUTO_TEST_SUITE_END()
```

---

## 异常测试

```cpp
#define BOOST_TEST_MODULE ExceptionTest
#include <boost/test/included/unit_test.hpp>
#include <stdexcept>

void throw_runtime_error() {
    throw std::runtime_error("错误信息");
}

void throw_nothing() {
    // 什么都不抛出
}

BOOST_AUTO_TEST_CASE(test_exceptions) {
    // 检查是否抛出异常
    BOOST_CHECK_THROW(throw_runtime_error(), std::runtime_error);
    
    // 检查不抛出异常
    BOOST_CHECK_NO_THROW(throw_nothing());
    
    // 检查特定异常消息
    BOOST_CHECK_EXCEPTION(
        throw_runtime_error(),
        std::runtime_error,
        [](const std::runtime_error& e) {
            return std::string(e.what()) == "错误信息";
        }
    );
}
```

---

## Fixture

```cpp
#define BOOST_TEST_MODULE FixtureTest
#include <boost/test/included/unit_test.hpp>
#include <vector>

struct VectorFixture {
    VectorFixture() {
        vec.push_back(1);
        vec.push_back(2);
        vec.push_back(3);
        BOOST_TEST_MESSAGE("设置 fixture");
    }
    
    ~VectorFixture() {
        BOOST_TEST_MESSAGE("清理 fixture");
    }
    
    std::vector<int> vec;
};

BOOST_FIXTURE_TEST_SUITE(VectorTests, VectorFixture)

BOOST_AUTO_TEST_CASE(test_size) {
    BOOST_CHECK_EQUAL(vec.size(), 3);
}

BOOST_AUTO_TEST_CASE(test_elements) {
    BOOST_CHECK_EQUAL(vec[0], 1);
    BOOST_CHECK_EQUAL(vec[1], 2);
    BOOST_CHECK_EQUAL(vec[2], 3);
}

BOOST_AUTO_TEST_SUITE_END()
```

---

## 参数化测试

```cpp
#define BOOST_TEST_MODULE ParameterizedTest
#include <boost/test/included/unit_test.hpp>
#include <boost/test/data/test_case.hpp>
#include <boost/test/data/monomorphic.hpp>

namespace bdata = boost::unit_test::data;

int square(int x) {
    return x * x;
}

BOOST_DATA_TEST_CASE(
    test_square,
    bdata::make({1, 2, 3, 4, 5}) ^ bdata::make({1, 4, 9, 16, 25}),
    input, expected
) {
    BOOST_TEST(square(input) == expected);
}
```

---

## 浮点数比较

```cpp
#define BOOST_TEST_MODULE FloatTest
#include <boost/test/included/unit_test.hpp>
#include <cmath>

BOOST_AUTO_TEST_CASE(test_floating_point) {
    double pi = 3.14159265359;
    
    // 相对误差（百分比）
    BOOST_CHECK_CLOSE(pi, 3.14159, 0.001);  // 0.001% 容差
    
    // 绝对误差
    BOOST_CHECK_SMALL(pi - 3.14159, 0.00001);
    
    // 使用容差
    double tolerance = 0.0001;
    BOOST_TEST(std::abs(pi - 3.14159) < tolerance);
}
```

---

## 集合比较

```cpp
#define BOOST_TEST_MODULE CollectionTest
#include <boost/test/included/unit_test.hpp>
#include <boost/test/tools/output_test_stream.hpp>
#include <vector>

BOOST_AUTO_TEST_CASE(test_collections) {
    std::vector<int> v1 = {1, 2, 3, 4, 5};
    std::vector<int> v2 = {1, 2, 3, 4, 5};
    std::vector<int> v3 = {1, 2, 3, 4, 6};
    
    // 逐元素比较
    BOOST_TEST(v1 == v2, boost::test_tools::per_element());
    BOOST_TEST(v1 != v3, boost::test_tools::per_element());
}
```

---

## 条件测试

```cpp
#define BOOST_TEST_MODULE ConditionalTest
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_CASE(test_conditional, 
    * boost::unit_test::enable_if<true>()) {
    BOOST_TEST(true);
}

BOOST_AUTO_TEST_CASE(test_disabled,
    * boost::unit_test::disabled()) {
    BOOST_TEST(false);  // 不会运行
}

BOOST_AUTO_TEST_CASE(test_expected_failures,
    * boost::unit_test::expected_failures(1)) {
    BOOST_TEST(false);  // 预期失败
}
```

---

## 性能测试

```cpp
#define BOOST_TEST_MODULE PerformanceTest
#include <boost/test/included/unit_test.hpp>
#include <chrono>
#include <vector>
#include <algorithm>

BOOST_AUTO_TEST_CASE(test_sorting_performance) {
    std::vector<int> data(10000);
    for (int i = 0; i < 10000; ++i) {
        data[i] = 10000 - i;
    }
    
    auto start = std::chrono::high_resolution_clock::now();
    std::sort(data.begin(), data.end());
    auto end = std::chrono::high_resolution_clock::now();
    
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    BOOST_TEST_MESSAGE("排序耗时: " << duration.count() << " ms");
    
    // 验证排序正确
    BOOST_TEST(std::is_sorted(data.begin(), data.end()));
}
```

---

## 自定义输出

```cpp
#define BOOST_TEST_MODULE CustomOutputTest
#include <boost/test/included/unit_test.hpp>

struct Point {
    int x, y;
    
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// 自定义输出
std::ostream& operator<<(std::ostream& os, const Point& p) {
    return os << "Point(" << p.x << ", " << p.y << ")";
}

BOOST_AUTO_TEST_CASE(test_custom_type) {
    Point p1{10, 20};
    Point p2{10, 20};
    Point p3{15, 25};
    
    BOOST_CHECK_EQUAL(p1, p2);
    BOOST_CHECK_NE(p1, p3);
}
```

---

## 超时测试

```cpp
#define BOOST_TEST_MODULE TimeoutTest
#include <boost/test/included/unit_test.hpp>
#include <thread>
#include <chrono>

BOOST_AUTO_TEST_CASE(test_fast,
    * boost::unit_test::timeout(1)) {  // 1秒超时
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    BOOST_TEST(true);
}

// 这个测试会超时
// BOOST_AUTO_TEST_CASE(test_slow,
//     * boost::unit_test::timeout(1)) {
//     std::this_thread::sleep_for(std::chrono::seconds(2));
//     BOOST_TEST(true);
// }
```

---

## 参考资源

- [Boost.Test 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/test/doc/html/index.html)
- [单元测试最佳实践](https://www.boost.org/doc/libs/1_90_0/libs/test/doc/html/boost_test/testing_tools.html)

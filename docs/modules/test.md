# Boost.Test - 单元测试框架

## 概述

Boost.Test 是一个功能丰富的 C++ 单元测试框架。

**类型**: 需要编译链接的库

**链接库**: `-lboost_unit_test_framework`

---

## 快速开始

```cpp
#define BOOST_TEST_MODULE MyTest
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_CASE(test_addition) {
    BOOST_TEST(2 + 2 == 4);
    BOOST_TEST(10 + 5 == 15);
}

BOOST_AUTO_TEST_CASE(test_subtraction) {
    BOOST_TEST(10 - 5 == 5);
    BOOST_TEST(100 - 1 == 99);
}
```

**编译**:
```bash
g++ -std=c++11 test.cpp -o test && ./test
```

---

## 断言

```cpp
#include <boost/test/unit_test.hpp>
#include <string>
#include <vector>

BOOST_AUTO_TEST_SUITE(assertion_tests)

BOOST_AUTO_TEST_CASE(test_comparisons) {
    // 相等性
    BOOST_TEST(42 == 42);
    BOOST_CHECK_EQUAL(42, 42);

    // 不等性
    BOOST_TEST(42 != 10);
    BOOST_CHECK_NE(42, 10);

    // 关系
    BOOST_TEST(10 < 20);
    BOOST_CHECK_LT(10, 20);
    BOOST_CHECK_LE(10, 10);
    BOOST_CHECK_GT(20, 10);
    BOOST_CHECK_GE(20, 20);
}

BOOST_AUTO_TEST_CASE(test_floats) {
    // 浮点数比较（带容差）
    BOOST_TEST(3.14 == 3.14, boost::test_tools::tolerance(0.001));
    BOOST_CHECK_CLOSE(3.14159, 3.14, 1.0); // 1% 容差
}

BOOST_AUTO_TEST_CASE(test_collections) {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {1, 2, 3};

    BOOST_TEST(v1 == v2);
}

BOOST_AUTO_TEST_CASE(test_exceptions) {
    // 检查异常
    BOOST_CHECK_THROW(throw std::runtime_error("error"), std::runtime_error);
    BOOST_CHECK_NO_THROW(int x = 42);
}

BOOST_AUTO_TEST_SUITE_END()
```

---

## 测试套件

```cpp
#include <boost/test/unit_test.hpp>

BOOST_AUTO_TEST_SUITE(math_tests)

BOOST_AUTO_TEST_CASE(test_add) {
    BOOST_TEST(2 + 2 == 4);
}

BOOST_AUTO_TEST_CASE(test_multiply) {
    BOOST_TEST(3 * 4 == 12);
}

BOOST_AUTO_TEST_SUITE_END()

BOOST_AUTO_TEST_SUITE(string_tests)

BOOST_AUTO_TEST_CASE(test_concat) {
    std::string s = "Hello" + std::string(" World");
    BOOST_TEST(s == "Hello World");
}

BOOST_AUTO_TEST_SUITE_END()
```

---

## Fixture（测试夹具）

```cpp
#include <boost/test/unit_test.hpp>
#include <vector>

struct VectorFixture {
    VectorFixture() {
        vec.push_back(1);
        vec.push_back(2);
        vec.push_back(3);
        std::cout << "Setup" << std::endl;
    }

    ~VectorFixture() {
        std::cout << "Teardown" << std::endl;
    }

    std::vector<int> vec;
};

BOOST_FIXTURE_TEST_SUITE(vector_tests, VectorFixture)

BOOST_AUTO_TEST_CASE(test_size) {
    BOOST_TEST(vec.size() == 3);
}

BOOST_AUTO_TEST_CASE(test_elements) {
    BOOST_TEST(vec[0] == 1);
    BOOST_TEST(vec[1] == 2);
    BOOST_TEST(vec[2] == 3);
}

BOOST_AUTO_TEST_SUITE_END()
```

---

## 参数化测试

```cpp
#include <boost/test/unit_test.hpp>
#include <boost/test/data/test_case.hpp>
#include <boost/test/data/monomorphic.hpp>

namespace data = boost::unit_test::data;

int square(int x) { return x * x; }

BOOST_DATA_TEST_CASE(
    test_squares,
    data::make({1, 2, 3, 4, 5}) ^
    data::make({1, 4, 9, 16, 25}),
    input, expected
) {
    BOOST_TEST(square(input) == expected);
}
```

---

## 最佳实践

1. **一个测试一个断言**: 保持测试简单
2. **使用有意义的测试名**: 描述测试内容
3. **独立测试**: 测试间不应有依赖
4. **使用 fixture**: 复用测试设置
5. **测试边界情况**: 包括异常输入

---

## 编译选项

```bash
# 使用 header-only
g++ -std=c++11 -DBOOST_TEST_DYN_LINK test.cpp -lboost_unit_test_framework

# CMake
find_package(Boost REQUIRED COMPONENTS unit_test_framework)
target_link_libraries(tests Boost::unit_test_framework)
```

---

## 参考资源

- [Boost.Test 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/test/doc/html/index.html)

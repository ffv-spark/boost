# Boost.Tuple - 元组库

## 概述

Boost.Tuple 提供固定大小的异构元素集合。

**类型**: 仅头文件库

**注意**: C++11 已引入 `std::tuple`

---

## 快速开始

```cpp
#include <boost/tuple/tuple.hpp>
#include <boost/tuple/tuple_io.hpp>
#include <iostream>
#include <string>

int main() {
    // 创建元组
    boost::tuple<int, std::string, double> person(25, "Alice", 5000.0);

    // 访问元素
    std::cout << "Age: " << boost::get<0>(person) << std::endl;
    std::cout << "Name: " << boost::get<1>(person) << std::endl;
    std::cout << "Salary: " << boost::get<2>(person) << std::endl;

    // 输出元组
    std::cout << "Tuple: " << person << std::endl;

    return 0;
}
```

---

## 元组操作

```cpp
#include <boost/tuple/tuple.hpp>
#include <boost/tuple/tuple_comparison.hpp>
#include <iostream>

int main() {
    boost::tuple<int, int> t1(1, 2);
    boost::tuple<int, int> t2(1, 3);

    // 比较
    std::cout << "t1 < t2: " << (t1 < t2) << std::endl;
    std::cout << "t1 == t2: " << (t1 == t2) << std::endl;

    // make_tuple
    auto t3 = boost::make_tuple(10, "Hello", 3.14);

    // tie
    int x;
    std::string s;
    double d;
    boost::tie(x, s, d) = t3;

    std::cout << "x=" << x << ", s=" << s << ", d=" << d << std::endl;

    return 0;
}
```

---

## 函数返回多值

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <string>

boost::tuple<bool, std::string, int> process_data() {
    return boost::make_tuple(true, "Success", 42);
}

int main() {
    bool success;
    std::string message;
    int result;

    boost::tie(success, message, result) = process_data();

    if (success) {
        std::cout << message << ": " << result << std::endl;
    }

    return 0;
}
```

---

## 与 std::tuple 对比

```cpp
#include <boost/tuple/tuple.hpp>
#include <tuple>
#include <iostream>

int main() {
    // Boost.Tuple
    auto b_t = boost::make_tuple(1, "hello", 3.14);
    std::cout << boost::get<0>(b_t) << std::endl;

    // std::tuple (C++11)
    auto s_t = std::make_tuple(1, "hello", 3.14);
    std::cout << std::get<0>(s_t) << std::endl;

    // C++11+ 推荐使用 std::tuple

    return 0;
}
```

---

## 参考资源

- [Boost.Tuple 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/tuple/doc/tuple_users_guide.html)

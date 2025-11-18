# Boost.Tuple - 元组库

## 概述

Boost.Tuple 提供固定大小的异构容器，可以存储不同类型的值。

**类型**: 仅头文件库

**注意**: C++11 引入了 std::tuple，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/tuple/tuple.hpp>
#include <boost/tuple/tuple_io.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::tuple;
    using boost::make_tuple;

    // 创建元组
    tuple<std::string, int, double> person("Alice", 30, 65.5);

    std::cout << "姓名: " << person.get<0>() << std::endl;
    std::cout << "年龄: " << person.get<1>() << std::endl;
    std::cout << "体重: " << person.get<2>() << "kg" << std::endl;

    // 使用 make_tuple
    auto data = make_tuple(42, 3.14, "hello");
    std::cout << "\n元组: " << data << std::endl;

    return 0;
}
```

---

## 创建元组

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::tuple;
    using boost::make_tuple;

    // 直接构造
    tuple<int, double, std::string> t1(42, 3.14, "hello");

    // 使用 make_tuple
    auto t2 = make_tuple(10, 20, 30);

    // 默认构造
    tuple<int, double> t3;  // (0, 0.0)

    // 拷贝构造
    tuple<int, double> t4(t3);

    std::cout << "t1: " << t1.get<0>() << ", "
              << t1.get<1>() << ", " << t1.get<2>() << std::endl;

    return 0;
}
```

---

## tie 函数

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <string>

// 返回多个值
boost::tuple<int, int, int> divide_with_remainder(int dividend, int divisor) {
    return boost::make_tuple(dividend / divisor, dividend % divisor, dividend);
}

int main() {
    using boost::tie;
    using boost::ignore;

    // 使用 tie 解包元组
    int quotient, remainder, original;
    tie(quotient, remainder, original) = divide_with_remainder(17, 5);

    std::cout << original << " / 5 = " << quotient
              << " 余 " << remainder << std::endl;

    // 忽略某些值
    int q;
    tie(q, ignore, ignore) = divide_with_remainder(20, 3);
    std::cout << "商: " << q << std::endl;

    return 0;
}
```

---

## 比较操作

```cpp
#include <boost/tuple/tuple.hpp>
#include <boost/tuple/tuple_comparison.hpp>
#include <iostream>

int main() {
    using boost::tuple;
    using boost::make_tuple;

    auto t1 = make_tuple(1, 2, 3);
    auto t2 = make_tuple(1, 2, 3);
    auto t3 = make_tuple(1, 2, 4);

    std::cout << std::boolalpha;
    std::cout << "t1 == t2: " << (t1 == t2) << std::endl;
    std::cout << "t1 != t3: " << (t1 != t3) << std::endl;
    std::cout << "t1 < t3: " << (t1 < t3) << std::endl;
    std::cout << "t1 <= t2: " << (t1 <= t2) << std::endl;

    return 0;
}
```

---

## 返回多个值

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <string>
#include <cmath>

// 返回平均值、最小值、最大值
boost::tuple<double, int, int> statistics(const int* arr, int size) {
    int min_val = arr[0];
    int max_val = arr[0];
    double sum = 0;

    for (int i = 0; i < size; ++i) {
        sum += arr[i];
        if (arr[i] < min_val) min_val = arr[i];
        if (arr[i] > max_val) max_val = arr[i];
    }

    return boost::make_tuple(sum / size, min_val, max_val);
}

int main() {
    using boost::tie;

    int numbers[] = {5, 2, 8, 1, 9, 3, 7};
    int size = sizeof(numbers) / sizeof(numbers[0]);

    double avg;
    int min_v, max_v;

    tie(avg, min_v, max_v) = statistics(numbers, size);

    std::cout << "平均值: " << avg << std::endl;
    std::cout << "最小值: " << min_v << std::endl;
    std::cout << "最大值: " << max_v << std::endl;

    return 0;
}
```

---

## 元组容器

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    using boost::tuple;
    using boost::make_tuple;

    // 元组的 vector
    std::vector<tuple<std::string, int, double>> students;

    students.push_back(make_tuple("Alice", 20, 85.5));
    students.push_back(make_tuple("Bob", 22, 90.0));
    students.push_back(make_tuple("Charlie", 21, 78.5));

    std::cout << "学生列表:\n";
    for (const auto& student : students) {
        std::cout << "  " << student.get<0>()
                  << ", " << student.get<1>() << "岁"
                  << ", 成绩: " << student.get<2>() << std::endl;
    }

    return 0;
}
```

---

## 与 std::pair 比较

```cpp
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <utility>

int main() {
    using boost::tuple;
    using boost::make_tuple;

    // std::pair 只能存储两个元素
    std::pair<int, std::string> p(42, "answer");

    // tuple 可以存储任意数量的元素
    auto t = make_tuple(42, "answer", 3.14, 'x');

    std::cout << "pair: (" << p.first << ", " << p.second << ")" << std::endl;
    std::cout << "tuple: (" << t.get<0>() << ", " << t.get<1>()
              << ", " << t.get<2>() << ", " << t.get<3>() << ")" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Tuple 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/tuple/doc/html/index.html)
- [std::tuple 参考](https://en.cppreference.com/w/cpp/utility/tuple)

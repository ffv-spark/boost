# Boost.Iterator - 迭代器库

## 概述

Boost.Iterator 提供迭代器工具和适配器。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/iterator/counting_iterator.hpp>
#include <boost/iterator/filter_iterator.hpp>
#include <boost/iterator/transform_iterator.hpp>
#include <iostream>
#include <vector>

int main() {
    // 计数迭代器
    std::cout << "Counting: ";
    for (auto it = boost::counting_iterator<int>(0);
         it != boost::counting_iterator<int>(10); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    // 过滤迭代器
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    auto is_even = [](int n) { return n % 2 == 0; };

    std::cout << "Even numbers: ";
    for (auto it = boost::make_filter_iterator(is_even, nums.begin(), nums.end());
         it != boost::make_filter_iterator(is_even, nums.end(), nums.end()); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    // 转换迭代器
    auto double_it = [](int n) { return n * 2; };

    std::cout << "Doubled: ";
    for (auto it = boost::make_transform_iterator(nums.begin(), double_it);
         it != boost::make_transform_iterator(nums.end(), double_it); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 计数迭代器

```cpp
#include <boost/iterator/counting_iterator.hpp>
#include <boost/range/iterator_range.hpp>
#include <iostream>
#include <algorithm>

int main() {
    // 生成 0-9 的序列
    auto range = boost::make_iterator_range(
        boost::counting_iterator<int>(0),
        boost::counting_iterator<int>(10)
    );

    std::cout << "Range: ";
    for (int x : range) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 函数输出迭代器

```cpp
#include <boost/iterator/function_output_iterator.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 创建函数输出迭代器
    auto print = [](int x) { std::cout << x << " "; };
    auto out_it = boost::make_function_output_iterator(print);

    // 复制到输出
    std::copy(nums.begin(), nums.end(), out_it);
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Iterator 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/iterator/doc/index.html)

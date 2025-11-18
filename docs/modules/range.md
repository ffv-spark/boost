# Boost.Range - 范围库

## 概述

Boost.Range 提供范围抽象和范围适配器，简化容器和算法操作。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/range/algorithm.hpp>
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 范围算法
    boost::sort(nums);

    // 范围适配器
    for (int x : nums | boost::adaptors::filtered([](int n) { return n % 2 == 0; })
                      | boost::adaptors::transformed([](int n) { return n * 2; })) {
        std::cout << x << " ";  // 输出: 4 8 12 16 20
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 范围适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

using namespace boost::adaptors;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 过滤
    std::cout << "Filtered (even): ";
    for (int x : nums | filtered([](int n) { return n % 2 == 0; })) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    // 转换
    std::cout << "Transformed (*2): ";
    for (int x : nums | transformed([](int n) { return n * 2; })) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    // 反转
    std::cout << "Reversed: ";
    for (int x : nums | reversed) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    // 切片
    std::cout << "Sliced [2, 5): ";
    for (int x : nums | sliced(2, 5)) {
        std::cout << x << " ";  // 3 4 5
    }
    std::cout << "\n";

    // 组合
    std::cout << "Combined: ";
    for (int x : nums | filtered([](int n) { return n > 5; })
                      | transformed([](int n) { return n * 2; })
                      | reversed) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 范围算法

```cpp
#include <boost/range/algorithm.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums = {5, 2, 8, 1, 9, 3};

    // 排序
    boost::sort(nums);

    // 查找
    auto it = boost::find(nums, 8);
    if (it != nums.end()) {
        std::cout << "Found: " << *it << std::endl;
    }

    // 统计
    int count = boost::count(nums, 2);
    std::cout << "Count of 2: " << count << std::endl;

    // 复制
    std::vector<int> nums2;
    boost::copy(nums, std::back_inserter(nums2));

    // 累加
    int sum = boost::accumulate(nums, 0);
    std::cout << "Sum: " << sum << std::endl;

    return 0;
}
```

---

## 范围转换

```cpp
#include <boost/range/algorithm.hpp>
#include <boost/range/adaptors.hpp>
#include <boost/range/numeric.hpp>
#include <iostream>
#include <vector>
#include <list>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 转换为其他容器
    std::list<int> lst;
    boost::copy(vec, std::back_inserter(lst));

    // 范围操作
    auto doubled = vec | boost::adaptors::transformed([](int n) { return n * 2; });

    std::cout << "Doubled: ";
    for (int x : doubled) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Range 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/range/doc/html/index.html)

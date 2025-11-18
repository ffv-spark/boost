# Boost.Array - 固定大小数组

## 概述

Boost.Array 提供固定大小的 STL 兼容数组。

**类型**: 仅头文件库

**注意**: C++11 已引入 `std::array`

---

## 快速开始

```cpp
#include <boost/array.hpp>
#include <iostream>

int main() {
    boost::array<int, 5> arr = {{1, 2, 3, 4, 5}};

    // 访问元素
    std::cout << "First: " << arr[0] << std::endl;
    std::cout << "Last: " << arr.back() << std::endl;

    // 遍历
    for (int x : arr) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 大小
    std::cout << "Size: " << arr.size() << std::endl;

    return 0;
}
```

---

## 与 std::array 对比

```cpp
#include <boost/array.hpp>
#include <array>
#include <iostream>

int main() {
    // Boost.Array
    boost::array<int, 3> b_arr = {{1, 2, 3}};

    // std::array (C++11)
    std::array<int, 3> s_arr = {1, 2, 3};

    // API 完全相同
    std::cout << "Boost size: " << b_arr.size() << std::endl;
    std::cout << "Std size: " << s_arr.size() << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Array 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/array.html)

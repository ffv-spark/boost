# Boost.Array - 固定大小数组库

## 概述

Boost.Array 提供固定大小的 STL 兼容数组容器，是 std::array 的前身。

**类型**: 仅头文件库

**注意**: C++11 引入了 std::array，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/array.hpp>
#include <iostream>

int main() {
    // 创建固定大小数组
    boost::array<int, 5> arr = {{1, 2, 3, 4, 5}};

    std::cout << "数组: ";
    for (int x : arr) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    std::cout << "大小: " << arr.size() << std::endl;
    std::cout << "第一个元素: " << arr.front() << std::endl;
    std::cout << "最后一个元素: " << arr.back() << std::endl;

    return 0;
}
```

---

## 元素访问

```cpp
#include <boost/array.hpp>
#include <iostream>

int main() {
    boost::array<int, 5> arr = {{10, 20, 30, 40, 50}};

    // 使用 [] 访问
    std::cout << "arr[2] = " << arr[2] << std::endl;

    // 使用 at() 访问（带边界检查）
    try {
        std::cout << "arr.at(3) = " << arr.at(3) << std::endl;
        // arr.at(10) = 0;  // 会抛出异常
    } catch (const std::out_of_range& e) {
        std::cout << "异常: " << e.what() << std::endl;
    }

    // 直接访问数据
    int* data = arr.data();
    std::cout << "通过指针访问: " << data[1] << std::endl;

    return 0;
}
```

---

## 迭代器

```cpp
#include <boost/array.hpp>
#include <iostream>
#include <algorithm>

int main() {
    boost::array<int, 5> arr = {{5, 2, 8, 1, 9}};

    std::cout << "原始数组: ";
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 排序
    std::sort(arr.begin(), arr.end());

    std::cout << "排序后: ";
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 填充和交换

```cpp
#include <boost/array.hpp>
#include <iostream>

int main() {
    boost::array<int, 5> arr1 = {{1, 2, 3, 4, 5}};
    boost::array<int, 5> arr2;

    // 填充
    arr2.fill(99);

    std::cout << "arr2 填充后: ";
    for (int x : arr2) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 交换
    arr1.swap(arr2);

    std::cout << "交换后 arr1: ";
    for (int x : arr1) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 多维数组

```cpp
#include <boost/array.hpp>
#include <iostream>

int main() {
    // 3x4 矩阵
    boost::array<boost::array<int, 4>, 3> matrix = {{
        {{1, 2, 3, 4}},
        {{5, 6, 7, 8}},
        {{9, 10, 11, 12}}
    }};

    std::cout << "矩阵:\n";
    for (size_t i = 0; i < matrix.size(); ++i) {
        for (size_t j = 0; j < matrix[i].size(); ++j) {
            std::cout << matrix[i][j] << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 与 C 数组比较

```cpp
#include <boost/array.hpp>
#include <iostream>
#include <algorithm>

int main() {
    // Boost.Array
    boost::array<int, 5> arr = {{1, 2, 3, 4, 5}};

    // C 数组
    int c_arr[5] = {1, 2, 3, 4, 5};

    // Boost.Array 的优势：
    // 1. 知道自己的大小
    std::cout << "arr 大小: " << arr.size() << std::endl;

    // 2. 可以直接赋值
    boost::array<int, 5> arr2 = arr;

    // 3. 可以直接比较
    std::cout << "arr == arr2: " << (arr == arr2) << std::endl;

    // 4. STL 兼容
    std::sort(arr.begin(), arr.end());

    return 0;
}
```

---

## 参考资源

- [Boost.Array 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/array.html)
- [std::array 参考](https://en.cppreference.com/w/cpp/container/array)

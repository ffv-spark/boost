# Boost.Sort - 排序算法库

## 概述

Boost.Sort 提供高性能排序算法，包括spreadsort、pdqsort等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/sort/spreadsort/spreadsort.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> data = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    
    // 使用 spreadsort
    boost::sort::spreadsort::spreadsort(data.begin(), data.end());
    
    std::cout << "排序后: ";
    for (int x : data) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Sort 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/sort/doc/html/index.html)

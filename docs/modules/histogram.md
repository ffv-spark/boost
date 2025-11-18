# Boost.Histogram - 直方图库

## 概述

Boost.Histogram 提供多维直方图工具，用于数据分析和统计。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/histogram.hpp>
#include <iostream>
#include <vector>

namespace bh = boost::histogram;

int main() {
    // 创建1D直方图，10个bins，范围[0, 10)
    auto hist = bh::make_histogram(bh::axis::regular<>(10, 0.0, 10.0));
    
    // 填充数据
    std::vector<double> data = {1.5, 2.3, 5.7, 8.1, 3.4, 6.2};
    for (double value : data) {
        hist(value);
    }
    
    // 打印直方图
    std::cout << "直方图:\\n";
    for (auto&& bin : indexed(hist)) {
        std::cout << "Bin " << bin.index() << ": " << *bin << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Histogram 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/histogram/doc/html/index.html)

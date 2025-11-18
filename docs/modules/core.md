# Boost.Core - 核心工具库

## 概述

Boost.Core 提供核心实用工具，包括addressof、ref、swap等基础设施。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/core/ref.hpp>
#include <iostream>
#include <algorithm>

void increment(int& x) {
    ++x;
}

int main() {
    int value = 10;
    
    // 使用 ref 传递引用
    auto ref_value = boost::ref(value);
    increment(ref_value);
    
    std::cout << "value: " << value << std::endl;  // 11
    
    return 0;
}
```

---

## 参考资源

- [Boost.Core 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/core/doc/html/index.html)

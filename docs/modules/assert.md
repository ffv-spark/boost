# Boost.Assert - 断言宏库

## 概述

Boost.Assert 提供增强的断言宏，支持自定义断言处理器。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/assert.hpp>
#include <iostream>

int divide(int a, int b) {
    BOOST_ASSERT(b != 0 && "除数不能为0");
    return a / b;
}

int main() {
    std::cout << "10 / 2 = " << divide(10, 2) << std::endl;
    
    // divide(10, 0);  // 触发断言
    
    return 0;
}
```

---

## 参考资源

- [Boost.Assert 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/assert/doc/html/assert.html)

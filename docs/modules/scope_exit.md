# Boost.ScopeExit - 作用域退出库

## 概述

Boost.ScopeExit 提供RAII风格的作用域退出处理，确保资源正确清理。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>

int main() {
    int* data = new int[10];
    
    BOOST_SCOPE_EXIT(&data) {
        delete[] data;
        std::cout << "自动清理内存" << std::endl;
    } BOOST_SCOPE_EXIT_END
    
    // 使用 data...
    data[0] = 42;
    
    // 退出作用域时自动执行清理代码
    return 0;
}
```

---

## 参考资源

- [Boost.ScopeExit 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/scope_exit/doc/html/index.html)

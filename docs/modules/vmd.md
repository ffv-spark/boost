# Boost.VMD - 可变参数宏数据库

## 概述

Boost.VMD (Variadic Macro Data) 提供可变参数宏处理工具，扩展预处理器功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/vmd/vmd.hpp>
#include <iostream>

#define MY_TUPLE (1, 2, 3, 4)

int main() {
    // VMD用于复杂的预处理器元编程
    std::cout << "VMD用于处理可变参数宏" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.VMD 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/vmd/doc/html/index.html)

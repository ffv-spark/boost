# Boost.Integer - 整数类型库

## 概述

Boost.Integer 提供整数类型选择工具，可以根据需要选择合适大小的整数类型。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/integer.hpp>
#include <iostream>

int main() {
    // 选择能存储至少100的最小有符号整数类型
    typedef boost::int_t<100>::least least_int_type;
    
    // 选择恰好16位的整数类型
    typedef boost::int_t<16>::exact int16_type;
    
    std::cout << "least_int_type 大小: " << sizeof(least_int_type) << " 字节" << std::endl;
    std::cout << "int16_type 大小: " << sizeof(int16_type) << " 字节" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Integer 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/integer/doc/html/index.html)

# Boost.SafeNumerics - 安全数值库

## 概述

Boost.SafeNumerics 提供安全的整数类型，自动检测溢出、除零等错误。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/safe_numerics/safe_integer.hpp>
#include <iostream>

using namespace boost::safe_numerics;

int main() {
    try {
        safe<int> a = 2147483647;  // INT_MAX
        safe<int> b = 1;
        
        // 这会检测到溢出
        safe<int> c = a + b;
        
        std::cout << "结果: " << c << std::endl;
    }
    catch (const std::exception& e) {
        std::cerr << "检测到溢出: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.SafeNumerics 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/safe_numerics/doc/html/index.html)

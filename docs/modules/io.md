# Boost.IO - I/O状态保存器

## 概述

Boost.IO 提供I/O状态保存和恢复工具，确保流状态不被意外修改。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/io/ios_state.hpp>
#include <iostream>
#include <iomanip>

int main() {
    std::cout << "默认: " << 3.14159 << std::endl;
    
    {
        boost::io::ios_precision_saver saver(std::cout);
        std::cout << std::setprecision(2) << std::fixed;
        std::cout << "修改后: " << 3.14159 << std::endl;
    }  // saver析构，恢复精度
    
    std::cout << "恢复: " << 3.14159 << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.IO 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/io/doc/html/index.html)

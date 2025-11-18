# Boost.StaticString - 静态字符串库

## 概述

Boost.StaticString 提供固定容量的字符串，不使用动态内存分配。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/static_string.hpp>
#include <iostream>

int main() {
    // 创建最大容量为100的静态字符串
    boost::static_string<100> str = "Hello, World!";
    
    std::cout << "字符串: " << str << std::endl;
    std::cout << "长度: " << str.size() << std::endl;
    std::cout << "容量: " << str.capacity() << std::endl;
    
    str += " How are you?";
    std::cout << "追加后: " << str << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.StaticString 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/static_string/doc/html/index.html)

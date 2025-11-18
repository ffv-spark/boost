# Boost.Tribool - 三态布尔库

## 概述

Boost.Tribool 提供三态布尔类型，支持true、false和indeterminate（不确定）三个状态。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

int main() {
    using boost::logic::tribool;
    using boost::logic::indeterminate;
    
    tribool t = true;
    tribool f = false;
    tribool u = indeterminate;
    
    std::cout << std::boolalpha;
    std::cout << "t: " << (t ? "true" : "false") << std::endl;
    std::cout << "f: " << (f ? "true" : "false") << std::endl;
    std::cout << "u is indeterminate: " << indeterminate(u) << std::endl;
    
    // 逻辑运算
    tribool result = t && u;
    std::cout << "true && indeterminate is indeterminate: " 
              << indeterminate(result) << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Tribool 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/tribool.html)

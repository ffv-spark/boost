# Boost.EnableIf - 条件编译工具

## 概述

Boost.EnableIf 提供SFINAE（替换失败不是错误）工具，用于模板条件编译。

**类型**: 仅头文件库

**注意**: C++11引入了std::enable_if，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/utility/enable_if.hpp>
#include <boost/type_traits/is_integral.hpp>
#include <iostream>

template <typename T>
typename boost::enable_if<boost::is_integral<T>, void>::type
print(T value) {
    std::cout << "整数: " << value << std::endl;
}

template <typename T>
typename boost::disable_if<boost::is_integral<T>, void>::type
print(T value) {
    std::cout << "非整数: " << value << std::endl;
}

int main() {
    print(42);      // 整数
    print(3.14);    // 非整数
    
    return 0;
}
```

---

## 参考资源

- [Boost.EnableIf 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/core/doc/html/core/enable_if.html)

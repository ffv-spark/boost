# Boost.Parameter - 命名参数库

## 概述

Boost.Parameter 提供命名参数支持，允许以任意顺序传递参数。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/parameter.hpp>
#include <iostream>
#include <string>

BOOST_PARAMETER_NAME(name)
BOOST_PARAMETER_NAME(age)
BOOST_PARAMETER_NAME(city)

template <typename Args>
void print_person(const Args& args) {
    std::cout << "姓名: " << args[_name] << std::endl;
    std::cout << "年龄: " << args[_age | 0] << std::endl;  // 默认值0
    std::cout << "城市: " << args[_city | "未知"] << std::endl;
}

int main() {
    // 可以以任意顺序传参
    print_person((_name = "Alice", _age = 30, _city = "Beijing"));
    print_person((_city = "Shanghai", _name = "Bob"));
    
    return 0;
}
```

---

## 参考资源

- [Boost.Parameter 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/parameter/doc/html/index.html)

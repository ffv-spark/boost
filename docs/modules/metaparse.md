# Boost.Metaparse - 编译期解析器

## 概述

Boost.Metaparse 提供编译期字符串解析工具，可以在编译时解析DSL。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/metaparse/string.hpp>
#include <boost/metaparse/int_.hpp>
#include <boost/metaparse/token.hpp>
#include <boost/metaparse/entire_input.hpp>
#include <boost/metaparse/build_parser.hpp>
#include <iostream>

int main() {
    using namespace boost::metaparse;
    
    // Metaparse用于编译期解析
    std::cout << "Metaparse: 编译期字符串解析库" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Metaparse 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/metaparse/doc/html/index.html)

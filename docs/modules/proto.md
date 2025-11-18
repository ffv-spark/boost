# Boost.Proto - 表达式模板库

## 概述

Boost.Proto 提供构建和操作表达式模板的工具，用于创建嵌入式领域特定语言（EDSL）。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/proto/proto.hpp>
#include <iostream>

namespace proto = boost::proto;

template <typename Expr>
void evaluate(const Expr& expr) {
    std::cout << "表达式值: " << proto::value(expr) << std::endl;
}

int main() {
    proto::terminal<int>::type const _1 = {1};
    proto::terminal<int>::type const _2 = {2};
    
    auto expr = _1 + _2;
    
    // 注意：这只是构建表达式树，不计算值
    std::cout << "创建了表达式模板" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Proto 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/proto.html)

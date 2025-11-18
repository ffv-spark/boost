# Boost.Context - 上下文切换库

## 概述

Boost.Context 提供用户级上下文切换，是实现协程和纤程的基础。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>

namespace ctx = boost::context;

int main() {
    int data = 0;
    
    ctx::continuation source = ctx::callcc(
        [&data](ctx::continuation&& sink) {
            data = 42;
            std::cout << "协程中: data = " << data << std::endl;
            return std::move(sink);
        }
    );
    
    std::cout << "主程序中: data = " << data << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_context`

---

## 参考资源

- [Boost.Context 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/context/doc/html/index.html)

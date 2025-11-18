# Boost.ThrowException - 异常抛出库

## 概述

Boost.ThrowException 提供统一的异常抛出接口，支持在禁用异常的环境中工作。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/throw_exception.hpp>
#include <stdexcept>
#include <iostream>

void risky_operation(int value) {
    if (value < 0) {
        BOOST_THROW_EXCEPTION(std::invalid_argument("值不能为负"));
    }
    std::cout << "处理值: " << value << std::endl;
}

int main() {
    try {
        risky_operation(10);
        risky_operation(-5);
    }
    catch (const std::exception& e) {
        std::cerr << "捕获异常: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.ThrowException 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/exception/doc/throw_exception.html)

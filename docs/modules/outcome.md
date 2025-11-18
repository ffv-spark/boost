# Boost.Outcome - 结果类型库

## 概述

Boost.Outcome 提供result和outcome类型，用于优雅的错误处理。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/outcome.hpp>
#include <iostream>

namespace outcome = BOOST_OUTCOME_V2_NAMESPACE;

outcome::result<int> divide(int a, int b) {
    if (b == 0) {
        return outcome::failure(std::errc::invalid_argument);
    }
    return a / b;
}

int main() {
    auto result = divide(10, 2);
    
    if (result.has_value()) {
        std::cout << "结果: " << result.value() << std::endl;
    } else {
        std::cerr << "错误: " << result.error().message() << std::endl;
    }
    
    auto bad_result = divide(10, 0);
    if (!bad_result) {
        std::cerr << "除以0失败" << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Outcome 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/outcome/doc/html/index.html)

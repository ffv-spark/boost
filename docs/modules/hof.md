# Boost.HOF - 高阶函数库

## 概述

Boost.HOF (Higher-Order Functions) 提供函数组合、柯里化等高阶函数工具。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/hof.hpp>
#include <iostream>

int add(int x, int y) {
    return x + y;
}

int multiply(int x, int y) {
    return x * y;
}

int main() {
    // 函数组合
    auto composed = BOOST_HOF_LIFT(multiply) * BOOST_HOF_LIFT(add);
    
    // (3 + 4) * 2
    int result = composed(3, 4, 2);
    std::cout << "结果: " << result << std::endl;
    
    return 0;
}
```

---

## 柯里化

```cpp
#include <boost/hof.hpp>
#include <iostream>

int add3(int x, int y, int z) {
    return x + y + z;
}

int main() {
    // 柯里化
    auto curried = boost::hof::curry(add3);
    
    auto partial1 = curried(1);
    auto partial2 = partial1(2);
    int result = partial2(3);
    
    std::cout << "1 + 2 + 3 = " << result << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.HOF 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/hof/doc/html/index.html)

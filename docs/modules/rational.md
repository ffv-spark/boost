# Boost.Rational - 有理数库

## 概述

Boost.Rational 提供有理数（分数）类型，支持精确的分数运算。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    using boost::rational;
    
    rational<int> r1(1, 2);  // 1/2
    rational<int> r2(1, 3);  // 1/3
    
    // 加法
    rational<int> sum = r1 + r2;
    std::cout << "1/2 + 1/3 = " << sum << std::endl;
    
    // 乘法
    rational<int> product = r1 * r2;
    std::cout << "1/2 * 1/3 = " << product << std::endl;
    
    return 0;
}
```

---

## 有理数运算

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    using boost::rational;
    
    rational<int> a(2, 3);   // 2/3
    rational<int> b(3, 4);   // 3/4
    
    std::cout << "a = " << a << std::endl;
    std::cout << "b = " << b << std::endl;
    
    std::cout << "a + b = " << (a + b) << std::endl;
    std::cout << "a - b = " << (a - b) << std::endl;
    std::cout << "a * b = " << (a * b) << std::endl;
    std::cout << "a / b = " << (a / b) << std::endl;
    
    // 比较
    std::cout << "a < b: " << (a < b) << std::endl;
    std::cout << "a > b: " << (a > b) << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Rational 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/rational/rational.html)

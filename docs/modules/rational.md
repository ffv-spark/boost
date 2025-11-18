# Boost.Rational - 有理数库

## 概述

Boost.Rational 提供有理数（分数）类型，支持精确的分数运算，避免浮点误差。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    typedef boost::rational<int> rational;

    rational a(1, 2);  // 1/2
    rational b(1, 3);  // 1/3

    rational sum = a + b;  // 1/2 + 1/3 = 5/6

    std::cout << a << " + " << b << " = " << sum << std::endl;

    return 0;
}
```

---

## 基本运算

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    typedef boost::rational<int> rational;

    rational a(3, 4);   // 3/4
    rational b(2, 5);   // 2/5

    // 四则运算
    std::cout << "加法: " << a << " + " << b << " = " << (a + b) << std::endl;
    std::cout << "减法: " << a << " - " << b << " = " << (a - b) << std::endl;
    std::cout << "乘法: " << a << " * " << b << " = " << (a * b) << std::endl;
    std::cout << "除法: " << a << " / " << b << " = " << (a / b) << std::endl;

    // 自动约分
    rational c(6, 9);   // 自动简化为 2/3
    std::cout << "\n6/9 简化为: " << c << std::endl;

    return 0;
}
```

---

## 分子分母访问

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    typedef boost::rational<int> rational;

    rational r(12, 18);  // 自动约简为 2/3

    std::cout << "原始: 12/18" << std::endl;
    std::cout << "约简后: " << r << std::endl;
    std::cout << "分子: " << r.numerator() << std::endl;
    std::cout << "分母: " << r.denominator() << std::endl;

    // 赋值
    r.assign(5, 7);
    std::cout << "\n新值: " << r << std::endl;

    return 0;
}
```

---

## 比较运算

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    typedef boost::rational<int> rational;

    rational a(1, 2);
    rational b(2, 4);  // 等于 1/2
    rational c(3, 4);

    std::cout << std::boolalpha;
    std::cout << "1/2 == 2/4: " << (a == b) << std::endl;
    std::cout << "1/2 != 3/4: " << (a != c) << std::endl;
    std::cout << "1/2 < 3/4: " << (a < c) << std::endl;
    std::cout << "3/4 > 1/2: " << (c > a) << std::endl;

    return 0;
}
```

---

## 与整数和浮点数转换

```cpp
#include <boost/rational.hpp>
#include <iostream>
#include <iomanip>

int main() {
    typedef boost::rational<int> rational;

    // 从整数构造
    rational r1(5);  // 5/1
    std::cout << "整数 5: " << r1 << std::endl;

    // 转换为浮点数
    rational r2(1, 3);
    double d = boost::rational_cast<double>(r2);
    std::cout << "1/3 转为浮点: " << std::setprecision(10) << d << std::endl;

    // 整数部分
    rational r3(7, 3);  // 2 又 1/3
    std::cout << "7/3 的整数部分: " << r3.numerator() / r3.denominator() << std::endl;

    return 0;
}
```

---

## 精确计算示例

```cpp
#include <boost/rational.hpp>
#include <iostream>
#include <iomanip>

int main() {
    typedef boost::rational<int> rational;

    // 使用浮点数的问题
    double d1 = 0.1 + 0.2;
    std::cout << std::setprecision(20);
    std::cout << "浮点: 0.1 + 0.2 = " << d1 << std::endl;

    // 使用有理数的精确计算
    rational r1(1, 10);  // 0.1
    rational r2(2, 10);  // 0.2
    rational sum = r1 + r2;

    std::cout << "有理数: 1/10 + 2/10 = " << sum << std::endl;
    std::cout << "转为浮点: " << boost::rational_cast<double>(sum) << std::endl;

    return 0;
}
```

---

## 连分数

```cpp
#include <boost/rational.hpp>
#include <iostream>
#include <vector>

typedef boost::rational<int> rational;

// 计算连分数展开
std::vector<int> continued_fraction(rational r) {
    std::vector<int> cf;

    while (r.denominator() != 0) {
        int a = r.numerator() / r.denominator();
        cf.push_back(a);

        if (r.numerator() % r.denominator() == 0) {
            break;
        }

        r = rational(r.denominator(), r.numerator() % r.denominator());
    }

    return cf;
}

int main() {
    rational r(22, 7);  // 圆周率的近似

    std::cout << "22/7 的连分数展开: [";
    auto cf = continued_fraction(r);
    for (size_t i = 0; i < cf.size(); ++i) {
        std::cout << cf[i];
        if (i < cf.size() - 1) std::cout << ", ";
    }
    std::cout << "]" << std::endl;

    return 0;
}
```

---

## 求和示例

```cpp
#include <boost/rational.hpp>
#include <iostream>
#include <vector>

int main() {
    typedef boost::rational<int> rational;

    // 计算 1 + 1/2 + 1/3 + 1/4 + ... + 1/10
    rational sum(0);

    for (int i = 1; i <= 10; ++i) {
        sum += rational(1, i);
    }

    std::cout << "调和级数前10项和: " << sum << std::endl;
    std::cout << "浮点值: " << boost::rational_cast<double>(sum) << std::endl;

    return 0;
}
```

---

## 埃及分数

```cpp
#include <boost/rational.hpp>
#include <iostream>
#include <vector>

typedef boost::rational<int> rational;

// 将分数分解为埃及分数（单位分数之和）
std::vector<int> egyptian_fraction(rational r) {
    std::vector<int> result;

    while (r.numerator() != 0) {
        int denom = (r.denominator() + r.numerator() - 1) / r.numerator();
        result.push_back(denom);
        r -= rational(1, denom);
    }

    return result;
}

int main() {
    rational r(5, 6);

    std::cout << "5/6 的埃及分数表示: ";
    auto ef = egyptian_fraction(r);

    for (size_t i = 0; i < ef.size(); ++i) {
        std::cout << "1/" << ef[i];
        if (i < ef.size() - 1) std::cout << " + ";
    }
    std::cout << std::endl;

    // 验证
    rational sum(0);
    for (int d : ef) {
        sum += rational(1, d);
    }
    std::cout << "验证: " << sum << std::endl;

    return 0;
}
```

---

## 使用长整型

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    // 使用 long long 支持更大的数
    typedef boost::rational<long long> rational;

    rational big(1000000000LL, 3);

    std::cout << "大数有理数: " << big << std::endl;
    std::cout << "分子: " << big.numerator() << std::endl;
    std::cout << "分母: " << big.denominator() << std::endl;

    return 0;
}
```

---

## 异常处理

```cpp
#include <boost/rational.hpp>
#include <iostream>

int main() {
    typedef boost::rational<int> rational;

    try {
        // 除以零会抛出异常
        rational r(1, 0);
    } catch (const boost::bad_rational& e) {
        std::cout << "错误: " << e.what() << std::endl;
    }

    try {
        // 除法运算中除以零
        rational a(1, 2);
        rational b(0, 1);
        rational c = a / b;
    } catch (const boost::bad_rational& e) {
        std::cout << "除法错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Rational 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/rational/rational.html)

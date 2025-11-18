# Boost.Multiprecision - 高精度数值计算库

## 概述

Boost.Multiprecision 提供任意精度的整数和浮点数运算，突破内置类型的限制。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    // 任意精度整数
    cpp_int big_num = 1;

    // 计算 100!
    for (int i = 2; i <= 100; ++i) {
        big_num *= i;
    }

    std::cout << "100! = " << big_num << std::endl;

    return 0;
}
```

---

## 大整数运算

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    cpp_int a("123456789012345678901234567890");
    cpp_int b("987654321098765432109876543210");

    std::cout << "a = " << a << std::endl;
    std::cout << "b = " << b << std::endl;

    // 基本运算
    std::cout << "\na + b = " << (a + b) << std::endl;
    std::cout << "a * b = " << (a * b) << std::endl;
    std::cout << "b - a = " << (b - a) << std::endl;
    std::cout << "b / a = " << (b / a) << std::endl;
    std::cout << "b % a = " << (b % a) << std::endl;

    // 幂运算
    cpp_int result = pow(cpp_int(2), 1000);
    std::cout << "\n2^1000 的位数: " << result.str().length() << std::endl;

    return 0;
}
```

---

## 高精度浮点数

```cpp
#include <boost/multiprecision/cpp_dec_float.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::multiprecision;

    // 50位精度的浮点数
    typedef number<cpp_dec_float<50>> float50;

    float50 pi = 0;

    // 计算π（莱布尼茨级数）
    for (int i = 0; i < 1000000; ++i) {
        float50 term = float50(1) / (2 * i + 1);
        if (i % 2 == 0) {
            pi += term;
        } else {
            pi -= term;
        }
    }
    pi *= 4;

    std::cout << std::setprecision(50);
    std::cout << "π ≈ " << pi << std::endl;

    return 0;
}
```

---

## GMP 后端

```cpp
#include <boost/multiprecision/gmp.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    // 使用 GMP 后端（更快）
    mpz_int a, b;

    a = 1;
    for (int i = 1; i <= 50; ++i) {
        a *= i;
    }

    std::cout << "50! = " << a << std::endl;

    // 斐波那契数列
    mpz_int fib_prev = 0, fib_curr = 1;
    for (int i = 0; i < 100; ++i) {
        mpz_int temp = fib_curr;
        fib_curr += fib_prev;
        fib_prev = temp;
    }

    std::cout << "第100个斐波那契数: " << fib_curr << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lgmp -o example
```

---

## 有理数

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    // 高精度有理数
    typedef number<rational_adaptor<cpp_int_backend<>>> cpp_rational;

    cpp_rational r1(1, 3);   // 1/3
    cpp_rational r2(1, 6);   // 1/6

    std::cout << "1/3 + 1/6 = " << (r1 + r2) << std::endl;
    std::cout << "1/3 * 1/6 = " << (r1 * r2) << std::endl;

    // 调和级数前100项
    cpp_rational sum = 0;
    for (int i = 1; i <= 100; ++i) {
        sum += cpp_rational(1, i);
    }

    std::cout << "\n调和级数前100项和: " << sum << std::endl;
    std::cout << "浮点近似: " << sum.convert_to<double>() << std::endl;

    return 0;
}
```

---

## 固定精度整数

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    // 256位整数
    typedef number<cpp_int_backend<256, 256, unsigned_magnitude, unchecked, void>> uint256_t;

    uint256_t large = 1;
    large <<= 200;  // 2^200

    std::cout << "2^200 = " << large << std::endl;
    std::cout << "位数: " << msb(large) + 1 << std::endl;

    return 0;
}
```

---

## 复数

```cpp
#include <boost/multiprecision/cpp_complex.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    // 高精度复数
    typedef number<cpp_dec_float<50>> float50;
    typedef number<cpp_complex_backend<50>> complex50;

    complex50 z1(1.0, 2.0);  // 1 + 2i
    complex50 z2(3.0, 4.0);  // 3 + 4i

    std::cout << "z1 = " << z1 << std::endl;
    std::cout << "z2 = " << z2 << std::endl;

    std::cout << "\nz1 + z2 = " << (z1 + z2) << std::endl;
    std::cout << "z1 * z2 = " << (z1 * z2) << std::endl;

    std::cout << "\n|z1| = " << abs(z1) << std::endl;
    std::cout << "arg(z1) = " << arg(z1) << std::endl;

    return 0;
}
```

---

## 数论函数

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <boost/multiprecision/miller_rabin.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    cpp_int n = 1000000007;

    // 素性测试
    if (miller_rabin_test(n, 25)) {
        std::cout << n << " 是素数（概率）" << std::endl;
    } else {
        std::cout << n << " 不是素数" << std::endl;
    }

    // 最大公约数
    cpp_int a = 48;
    cpp_int b = 18;
    std::cout << "gcd(48, 18) = " << gcd(a, b) << std::endl;

    // 最小公倍数
    std::cout << "lcm(48, 18) = " << lcm(a, b) << std::endl;

    return 0;
}
```

---

## RSA 加密示例

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <boost/multiprecision/miller_rabin.hpp>
#include <iostream>

using namespace boost::multiprecision;

cpp_int power_mod(cpp_int base, cpp_int exp, cpp_int mod) {
    cpp_int result = 1;
    base %= mod;

    while (exp > 0) {
        if (exp % 2 == 1) {
            result = (result * base) % mod;
        }
        base = (base * base) % mod;
        exp /= 2;
    }

    return result;
}

int main() {
    // 简化的RSA示例（实际使用需要更大的素数）
    cpp_int p = 61;
    cpp_int q = 53;
    cpp_int n = p * q;
    cpp_int phi = (p - 1) * (q - 1);

    cpp_int e = 17;  // 公钥指数
    cpp_int d = 2753;  // 私钥指数（需要满足 e*d ≡ 1 (mod φ)）

    // 明文
    cpp_int message = 123;
    std::cout << "明文: " << message << std::endl;

    // 加密: c = m^e mod n
    cpp_int ciphertext = power_mod(message, e, n);
    std::cout << "密文: " << ciphertext << std::endl;

    // 解密: m = c^d mod n
    cpp_int decrypted = power_mod(ciphertext, d, n);
    std::cout << "解密: " << decrypted << std::endl;

    return 0;
}
```

---

## 性能比较

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>
#include <chrono>

int main() {
    using namespace boost::multiprecision;

    const int iterations = 1000;

    // 测试普通整数
    auto start1 = std::chrono::high_resolution_clock::now();
    long long result1 = 1;
    for (int i = 1; i <= 20; ++i) {
        result1 *= i;
    }
    auto end1 = std::chrono::high_resolution_clock::now();

    // 测试大整数
    auto start2 = std::chrono::high_resolution_clock::now();
    cpp_int result2 = 1;
    for (int i = 1; i <= 100; ++i) {
        result2 *= i;
    }
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration_cast<std::chrono::nanoseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::microseconds>(end2 - start2);

    std::cout << "20! 用 long long: " << duration1.count() << " 纳秒" << std::endl;
    std::cout << "100! 用 cpp_int: " << duration2.count() << " 微秒" << std::endl;

    return 0;
}
```

---

## 转换

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <boost/multiprecision/cpp_dec_float.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::multiprecision;

    // 从字符串构造
    cpp_int big("123456789012345678901234567890");
    std::cout << "从字符串: " << big << std::endl;

    // 转换为字符串
    std::string str = big.str();
    std::cout << "字符串长度: " << str.length() << std::endl;

    // 转换为十六进制
    std::cout << "十六进制: " << std::hex << big << std::dec << std::endl;

    // 整数转浮点
    cpp_dec_float_50 f = big.convert_to<cpp_dec_float_50>();
    std::cout << "浮点表示: " << f << std::endl;

    return 0;
}
```

---

## 位操作

```cpp
#include <boost/multiprecision/cpp_int.hpp>
#include <iostream>

int main() {
    using namespace boost::multiprecision;

    cpp_int a = 0xFF00;
    cpp_int b = 0x0F0F;

    std::cout << std::hex;
    std::cout << "a = 0x" << a << std::endl;
    std::cout << "b = 0x" << b << std::endl;

    std::cout << "a & b = 0x" << (a & b) << std::endl;
    std::cout << "a | b = 0x" << (a | b) << std::endl;
    std::cout << "a ^ b = 0x" << (a ^ b) << std::endl;
    std::cout << "~a = 0x" << (~a & 0xFFFF) << std::endl;

    std::cout << std::dec;
    std::cout << "\na << 4 = " << (a << 4) << std::endl;
    std::cout << "a >> 4 = " << (a >> 4) << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Multiprecision 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/multiprecision/doc/html/index.html)
- [后端类型](https://www.boost.org/doc/libs/1_90_0/libs/multiprecision/doc/html/boost_multiprecision/tut/ints.html)

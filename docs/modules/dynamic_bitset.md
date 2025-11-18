# Boost.DynamicBitset - 动态位集库

## 概述

Boost.DynamicBitset 提供动态大小的位集，类似于 std::bitset 但大小可在运行时确定。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    // 创建10位的位集
    boost::dynamic_bitset<> bits(10);

    // 设置某些位
    bits[0] = 1;
    bits[3] = 1;
    bits[7] = 1;

    std::cout << "位集: " << bits << std::endl;
    std::cout << "大小: " << bits.size() << std::endl;
    std::cout << "设置的位数: " << bits.count() << std::endl;

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(8);

    // 设置位
    bits.set(0);      // 设置第0位
    bits.set(2, 1);   // 设置第2位为1
    bits.set(5);

    std::cout << "设置后: " << bits << std::endl;

    // 重置位
    bits.reset(2);    // 清除第2位
    std::cout << "重置后: " << bits << std::endl;

    // 翻转位
    bits.flip(0);     // 翻转第0位
    std::cout << "翻转后: " << bits << std::endl;

    // 全部设置/重置/翻转
    bits.set();       // 全部设为1
    std::cout << "全设置: " << bits << std::endl;

    bits.reset();     // 全部清零
    std::cout << "全重置: " << bits << std::endl;

    bits.flip();      // 全部翻转
    std::cout << "全翻转: " << bits << std::endl;

    return 0;
}
```

---

## 位运算

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> a(8, 0b10101010);  // 170
    boost::dynamic_bitset<> b(8, 0b11001100);  // 204

    std::cout << "a:     " << a << std::endl;
    std::cout << "b:     " << b << std::endl;

    // 按位与
    std::cout << "a & b: " << (a & b) << std::endl;

    // 按位或
    std::cout << "a | b: " << (a | b) << std::endl;

    // 按位异或
    std::cout << "a ^ b: " << (a ^ b) << std::endl;

    // 按位取反
    std::cout << "~a:    " << (~a) << std::endl;

    // 复合赋值
    a &= b;
    std::cout << "a &= b: " << a << std::endl;

    return 0;
}
```

---

## 移位操作

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(8, 0b00001111);

    std::cout << "原始:   " << bits << std::endl;

    // 左移
    std::cout << "<<2:    " << (bits << 2) << std::endl;

    // 右移
    std::cout << ">>2:    " << (bits >> 2) << std::endl;

    // 复合赋值
    bits <<= 3;
    std::cout << "<<=3:   " << bits << std::endl;

    bits >>= 1;
    std::cout << ">>=1:   " << bits << std::endl;

    return 0;
}
```

---

## 动态调整大小

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(5);
    bits.set(2);
    bits.set(4);

    std::cout << "原始 (5位): " << bits << std::endl;

    // 扩展大小
    bits.resize(10);
    std::cout << "扩展到10位: " << bits << std::endl;

    // 缩小大小
    bits.resize(3);
    std::cout << "缩小到3位:  " << bits << std::endl;

    // 扩展时用1填充
    bits.resize(8, true);
    std::cout << "扩展用1填:  " << bits << std::endl;

    return 0;
}
```

---

## 转换为数值

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(8, 0b10110011);

    std::cout << "位集: " << bits << std::endl;

    // 转换为 unsigned long
    unsigned long value = bits.to_ulong();
    std::cout << "十进制: " << value << std::endl;
    std::cout << "十六进制: 0x" << std::hex << value << std::endl;

    // 转换为字符串
    std::string str = boost::to_string(bits);
    std::cout << "字符串: " << str << std::endl;

    return 0;
}
```

---

## 从字符串构造

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>
#include <string>

int main() {
    // 从字符串构造
    std::string binary = "10101100";
    boost::dynamic_bitset<> bits(binary);

    std::cout << "从字符串: " << bits << std::endl;

    // 部分字符串
    std::string partial = "11110000";
    boost::dynamic_bitset<> bits2(partial, 0, 4);  // 只取前4位

    std::cout << "部分字符串: " << bits2 << std::endl;

    return 0;
}
```

---

## 位查询

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(10);
    bits.set(1);
    bits.set(3);
    bits.set(5);
    bits.set(7);

    std::cout << "位集: " << bits << std::endl;

    // 查询信息
    std::cout << "大小: " << bits.size() << std::endl;
    std::cout << "设置的位数: " << bits.count() << std::endl;
    std::cout << "是否全为1: " << bits.all() << std::endl;
    std::cout << "是否有1: " << bits.any() << std::endl;
    std::cout << "是否全为0: " << bits.none() << std::endl;

    // 测试特定位
    std::cout << "第3位: " << bits.test(3) << std::endl;
    std::cout << "第4位: " << bits.test(4) << std::endl;

    return 0;
}
```

---

## 查找位

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> bits(20);
    bits.set(3);
    bits.set(7);
    bits.set(15);

    std::cout << "位集: " << bits << std::endl;

    // 查找第一个设置的位
    size_t pos = bits.find_first();
    if (pos != boost::dynamic_bitset<>::npos) {
        std::cout << "第一个1的位置: " << pos << std::endl;
    }

    // 查找下一个设置的位
    pos = bits.find_next(pos);
    while (pos != boost::dynamic_bitset<>::npos) {
        std::cout << "下一个1的位置: " << pos << std::endl;
        pos = bits.find_next(pos);
    }

    return 0;
}
```

---

## 子集检查

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>

int main() {
    boost::dynamic_bitset<> set1(8, 0b11110000);
    boost::dynamic_bitset<> set2(8, 0b01100000);
    boost::dynamic_bitset<> set3(8, 0b10001000);

    std::cout << "set1: " << set1 << std::endl;
    std::cout << "set2: " << set2 << std::endl;
    std::cout << "set3: " << set3 << std::endl;

    // 检查是否为子集
    std::cout << "\nset2 是 set1 的子集: "
              << ((set2 & set1) == set2) << std::endl;

    std::cout << "set3 是 set1 的子集: "
              << ((set3 & set1) == set3) << std::endl;

    // 检查是否相交
    std::cout << "set1 和 set3 相交: "
              << ((set1 & set3).any()) << std::endl;

    return 0;
}
```

---

## 压缩存储

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>
#include <vector>

int main() {
    // 使用位集压缩存储布尔数组
    const int size = 10000;

    // 普通 bool 数组
    std::vector<bool> bool_array(size);
    for (int i = 0; i < size; i += 2) {
        bool_array[i] = true;
    }

    // 使用 dynamic_bitset
    boost::dynamic_bitset<> bitset(size);
    for (int i = 0; i < size; i += 2) {
        bitset.set(i);
    }

    std::cout << "bool 数组大小: ~" << sizeof(bool) * size << " 字节" << std::endl;
    std::cout << "bitset 大小: ~" << bitset.num_blocks() * sizeof(boost::dynamic_bitset<>::block_type)
              << " 字节" << std::endl;

    std::cout << "\n设置的位数: " << bitset.count() << std::endl;

    return 0;
}
```

---

## 应用：素数筛

```cpp
#include <boost/dynamic_bitset.hpp>
#include <iostream>
#include <vector>

std::vector<int> sieve_of_eratosthenes(int n) {
    boost::dynamic_bitset<> is_prime(n + 1);
    is_prime.set();  // 全部设为 true
    is_prime[0] = is_prime[1] = 0;  // 0 和 1 不是素数

    for (int i = 2; i * i <= n; ++i) {
        if (is_prime[i]) {
            for (int j = i * i; j <= n; j += i) {
                is_prime[j] = 0;
            }
        }
    }

    std::vector<int> primes;
    for (size_t i = 2; i <= n; ++i) {
        if (is_prime[i]) {
            primes.push_back(i);
        }
    }

    return primes;
}

int main() {
    int n = 100;
    auto primes = sieve_of_eratosthenes(n);

    std::cout << n << " 以内的素数:\n";
    for (int p : primes) {
        std::cout << p << " ";
    }
    std::cout << "\n\n共 " << primes.size() << " 个素数" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.DynamicBitset 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/dynamic_bitset/dynamic_bitset.html)

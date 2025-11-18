# Boost.Typeof - typeof模拟库

## 概述

Boost.Typeof 提供C++03中的typeof功能模拟，是C++11 decltype的前身。

**类型**: 仅头文件库

**注意**: C++11引入了decltype和auto，优先使用标准特性

---

## 快速开始

```cpp
#include <boost/typeof/typeof.hpp>
#include <iostream>
#include <vector>

BOOST_AUTO(multiply, (int x, int y) { return x * y; });

int main() {
    std::vector<int> vec = {1, 2, 3};
    
    // 使用BOOST_AUTO自动推导类型
    BOOST_AUTO(it, vec.begin());
    
    std::cout << "第一个元素: " << *it << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Typeof 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/typeof.html)

# Boost.NumericConversion - 数值转换库

## 概述

Boost.NumericConversion 提供安全的数值类型转换，检测溢出和精度损失。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/numeric/conversion/cast.hpp>
#include <iostream>

int main() {
    try {
        double d = 123.45;
        
        // 安全转换为int
        int i = boost::numeric_cast<int>(d);
        std::cout << "转换成功: " << i << std::endl;
        
        // 尝试转换会溢出的值
        double large = 1e100;
        int overflow = boost::numeric_cast<int>(large);  // 抛出异常
    }
    catch (const boost::numeric::bad_numeric_cast& e) {
        std::cerr << "转换错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.NumericConversion 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/numeric/conversion/doc/html/index.html)

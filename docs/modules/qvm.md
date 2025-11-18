# Boost.QVM - 四元数/向量/矩阵库

## 概述

Boost.QVM 提供四元数、向量和矩阵的操作，常用于3D图形和物理模拟。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/qvm/vec.hpp>
#include <boost/qvm/vec_operations.hpp>
#include <iostream>

namespace qvm = boost::qvm;

int main() {
    // 创建3D向量
    qvm::vec<float, 3> v1 = {1.0f, 2.0f, 3.0f};
    qvm::vec<float, 3> v2 = {4.0f, 5.0f, 6.0f};
    
    // 向量加法
    auto v3 = v1 + v2;
    
    std::cout << "v3 = (" << v3.a[0] << ", " << v3.a[1] << ", " << v3.a[2] << ")" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.QVM 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/qvm/doc/index.html)

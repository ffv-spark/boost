# Boost.Align - 内存对齐库

## 概述

Boost.Align 提供内存对齐工具，确保数据正确对齐以提高性能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/align/aligned_alloc.hpp>
#include <iostream>

int main() {
    using namespace boost::alignment;
    
    // 分配16字节对齐的内存
    void* ptr = aligned_alloc(16, 1024);
    
    if (ptr) {
        std::cout << "成功分配16字节对齐的内存" << std::endl;
        aligned_free(ptr);
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Align 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/align.html)

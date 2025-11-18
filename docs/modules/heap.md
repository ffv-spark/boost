# Boost.Heap - 堆算法库

## 概述

Boost.Heap 提供多种堆数据结构，包括优先队列的高级实现。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/heap/fibonacci_heap.hpp>
#include <iostream>

int main() {
    boost::heap::fibonacci_heap<int> heap;
    
    heap.push(5);
    heap.push(2);
    heap.push(8);
    heap.push(1);
    
    std::cout << "堆顶元素: " << heap.top() << std::endl;
    
    heap.pop();
    std::cout << "弹出后堆顶: " << heap.top() << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Heap 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/heap.html)

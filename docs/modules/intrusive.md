# Boost.Intrusive - 侵入式容器库

## 概述

Boost.Intrusive 提供侵入式容器，不需要额外的内存分配，适用于性能敏感的场景。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/intrusive/list.hpp>
#include <iostream>

class MyClass : public boost::intrusive::list_base_hook<> {
public:
    int value;
    MyClass(int v) : value(v) {}
};

int main() {
    typedef boost::intrusive::list<MyClass> List;
    
    MyClass obj1(10), obj2(20), obj3(30);
    
    List list;
    list.push_back(obj1);
    list.push_back(obj2);
    list.push_back(obj3);
    
    for (const auto& item : list) {
        std::cout << item.value << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Intrusive 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/intrusive.html)

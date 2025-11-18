# Boost.Conversion - 类型转换库

## 概述

Boost.Conversion 提供安全的类型转换工具，包括polymorphic_cast、polymorphic_downcast等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/cast.hpp>
#include <iostream>

class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void hello() { std::cout << "Hello from Derived" << std::endl; }
};

int main() {
    Base* base = new Derived();
    
    // 安全的向下转换
    Derived* derived = boost::polymorphic_downcast<Derived*>(base);
    derived->hello();
    
    delete base;
    return 0;
}
```

---

## 参考资源

- [Boost.Conversion 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/conversion/cast.htm)

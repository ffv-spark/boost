# Boost.Utility - 通用工具库

## 概述

Boost.Utility 提供各种小型实用工具，包括noncopyable、checked_delete等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/utility.hpp>
#include <boost/noncopyable.hpp>
#include <iostream>

class Singleton : private boost::noncopyable {
public:
    static Singleton& instance() {
        static Singleton inst;
        return inst;
    }
    
    void doSomething() {
        std::cout << "Singleton 工作中" << std::endl;
    }
    
private:
    Singleton() = default;
};

int main() {
    Singleton::instance().doSomething();
    
    // Singleton copy = Singleton::instance();  // 编译错误：不可拷贝
    
    return 0;
}
```

---

## 参考资源

- [Boost.Utility 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/utility/utility.htm)

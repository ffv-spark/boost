# Boost.Move - 移动语义库

## 概述

Boost.Move 提供C++03中的移动语义模拟，是C++11移动语义的前身。

**类型**: 仅头文件库

**注意**: C++11引入了原生移动语义，优先使用标准特性

---

## 快速开始

```cpp
#include <boost/move/move.hpp>
#include <iostream>
#include <vector>

class MyClass {
private:
    BOOST_COPYABLE_AND_MOVABLE(MyClass)
    std::vector<int> data_;
    
public:
    MyClass() : data_(1000) {}
    
    // 拷贝构造
    MyClass(const MyClass& other) : data_(other.data_) {
        std::cout << "拷贝构造" << std::endl;
    }
    
    // 移动构造
    MyClass(BOOST_RV_REF(MyClass) other) 
        : data_(boost::move(other.data_)) {
        std::cout << "移动构造" << std::endl;
    }
};

int main() {
    MyClass obj1;
    MyClass obj2 = boost::move(obj1);  // 移动
    
    return 0;
}
```

---

## 参考资源

- [Boost.Move 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/move.html)

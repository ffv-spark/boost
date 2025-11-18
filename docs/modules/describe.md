# Boost.Describe - 反射描述库

## 概述

Boost.Describe 提供简单的反射支持，可以在编译时获取类型信息。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/describe.hpp>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
    double height;
};

BOOST_DESCRIBE_STRUCT(Person, (), (name, age, height))

int main() {
    using namespace boost::describe;
    
    Person p{"Alice", 30, 165.5};
    
    std::cout << "Person 字段:\\n";
    
    // 遍历所有字段
    using members = describe_members<Person, mod_any_access>;
    boost::mp11::mp_for_each<members>([&](auto D) {
        std::cout << "  " << D.name << ": ";
        
        if constexpr (std::is_same_v<typename decltype(D)::type, std::string>) {
            std::cout << p.*D.pointer << std::endl;
        } else if constexpr (std::is_same_v<typename decltype(D)::type, int>) {
            std::cout << p.*D.pointer << std::endl;
        } else if constexpr (std::is_same_v<typename decltype(D)::type, double>) {
            std::cout << p.*D.pointer << std::endl;
        }
    });
    
    return 0;
}
```

---

## 参考资源

- [Boost.Describe 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/describe/doc/html/index.html)

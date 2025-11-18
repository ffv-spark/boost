# Boost.Pool - 内存池库

## 概述

Boost.Pool 提供快速的内存池分配器，减少动态内存分配的开销。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/pool/pool.hpp>
#include <iostream>

int main() {
    // 创建int的内存池
    boost::pool<> p(sizeof(int));
    
    // 分配内存
    int* ptr1 = static_cast<int*>(p.malloc());
    int* ptr2 = static_cast<int*>(p.malloc());
    
    *ptr1 = 42;
    *ptr2 = 100;
    
    std::cout << "ptr1: " << *ptr1 << std::endl;
    std::cout << "ptr2: " << *ptr2 << std::endl;
    
    // 释放内存
    p.free(ptr1);
    p.free(ptr2);
    
    return 0;
}
```

---

## 对象池

```cpp
#include <boost/pool/object_pool.hpp>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
    
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "构造 " << name << std::endl;
    }
    
    ~Person() {
        std::cout << "析构 " << name << std::endl;
    }
};

int main() {
    boost::object_pool<Person> pool;
    
    // 构造对象
    Person* p1 = pool.construct("Alice", 30);
    Person* p2 = pool.construct("Bob", 25);
    
    std::cout << p1->name << ": " << p1->age << std::endl;
    std::cout << p2->name << ": " << p2->age << std::endl;
    
    // 析构对象
    pool.destroy(p1);
    pool.destroy(p2);
    
    return 0;
}
```

---

## 参考资源

- [Boost.Pool 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/pool/doc/html/index.html)

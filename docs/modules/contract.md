# Boost.Contract - 契约编程库

## 概述

Boost.Contract 提供契约编程支持，包括前置条件、后置条件和类不变式。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/contract.hpp>
#include <iostream>

void push(int* array, int& size, int value) {
    boost::contract::check c = boost::contract::function()
        .precondition([&] {
            BOOST_CONTRACT_ASSERT(size < 10);  // 前置条件
        })
        .postcondition([&] {
            BOOST_CONTRACT_ASSERT(size > 0);   // 后置条件
        })
    ;
    
    array[size++] = value;
}

int main() {
    int array[10];
    int size = 0;
    
    push(array, size, 42);
    std::cout << "添加元素，大小: " << size << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_contract`

---

## 类不变式

```cpp
#include <boost/contract.hpp>
#include <iostream>

class Counter {
public:
    Counter() : value_(0) {
        boost::contract::check c = boost::contract::constructor(this)
            .postcondition([&] {
                BOOST_CONTRACT_ASSERT(value_ >= 0);
            })
        ;
    }
    
    void increment() {
        boost::contract::check c = boost::contract::public_function(this)
            .precondition([&] {
                BOOST_CONTRACT_ASSERT(value_ >= 0);
            })
            .postcondition([&] {
                BOOST_CONTRACT_ASSERT(value_ > 0);
            })
        ;
        
        ++value_;
    }
    
    int get() const { return value_; }
    
private:
    int value_;
};

int main() {
    Counter counter;
    counter.increment();
    std::cout << "计数器值: " << counter.get() << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Contract 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/contract/doc/html/index.html)

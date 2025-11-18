# Boost.CallableTraits - 可调用对象特性库

## 概述

Boost.CallableTraits 提供编译期可调用对象（函数、函数对象、lambda）的特性查询。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/callable_traits.hpp>
#include <iostream>
#include <type_traits>

int func(int x, double y) {
    return static_cast<int>(x + y);
}

int main() {
    using namespace boost::callable_traits;
    
    // 获取参数数量
    constexpr auto arg_count = args_t<decltype(func)>::size();
    std::cout << "参数数量: " << arg_count << std::endl;
    
    // 获取返回类型
    using return_type = return_type_t<decltype(func)>;
    std::cout << "返回类型是int: " 
              << std::is_same<return_type, int>::value << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.CallableTraits 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/callable_traits/doc/html/index.html)

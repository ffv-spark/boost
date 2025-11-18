# Boost.Coroutine - 协程库

## 概述

Boost.Coroutine 提供协程支持，允许函数暂停和恢复执行。

**类型**: 需要编译的库

**注意**: Boost.Coroutine 已被 Coroutine2 替代

---

## 快速开始

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>

int main() {
    using namespace boost::coroutines2;
    
    coroutine<int>::pull_type source(
        [](coroutine<int>::push_type& sink) {
            for (int i = 0; i < 5; ++i) {
                sink(i);
            }
        }
    );
    
    for (int value : source) {
        std::cout << value << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_context`

---

## 生成器

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>

int main() {
    using namespace boost::coroutines2;
    
    coroutine<int>::pull_type numbers(
        [](coroutine<int>::push_type& yield) {
            int n = 0;
            while (true) {
                yield(n++);
            }
        }
    );
    
    std::cout << "前10个数字: ";
    for (int i = 0; i < 10; ++i) {
        std::cout << numbers.get() << " ";
        numbers();
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 斐波那契数列

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>

int main() {
    using namespace boost::coroutines2;
    
    coroutine<int>::pull_type fibonacci(
        [](coroutine<int>::push_type& yield) {
            int a = 0, b = 1;
            yield(a);
            yield(b);
            
            while (true) {
                int next = a + b;
                yield(next);
                a = b;
                b = next;
            }
        }
    );
    
    std::cout << "斐波那契数列: ";
    for (int i = 0; i < 10; ++i) {
        std::cout << fibonacci.get() << " ";
        fibonacci();
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Coroutine2 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/coroutine2/doc/html/index.html)

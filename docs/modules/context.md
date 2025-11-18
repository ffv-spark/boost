# Boost.Context - 上下文切换库

## 概述

Boost.Context 提供底层的执行上下文切换机制，是 Coroutine 和 Fiber 的基础。

**类型**: 需要编译链接的库

**链接库**: `-lboost_context`

---

## 快速开始

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>

namespace ctx = boost::context;

int main() {
    int data = 0;

    ctx::continuation c = ctx::callcc(
        [&data](ctx::continuation&& sink) {
            std::cout << "第一次进入上下文" << std::endl;
            data = 42;
            sink = sink.resume();

            std::cout << "第二次进入上下文" << std::endl;
            data = 100;

            return std::move(sink);
        }
    );

    std::cout << "数据: " << data << std::endl;  // 42

    c = c.resume();
    std::cout << "数据: " << data << std::endl;  // 100

    return 0;
}
```

**编译**:
```bash
g++ -std=c++14 example.cpp -lboost_context -o example
```

---

## 执行上下文切换

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>

namespace ctx = boost::context;

int main() {
    int counter = 0;

    ctx::continuation source = ctx::callcc(
        [&counter](ctx::continuation&& sink) {
            while (true) {
                std::cout << "上下文中，计数: " << ++counter << std::endl;
                sink = sink.resume();
            }
            return std::move(sink);
        }
    );

    for (int i = 0; i < 5; ++i) {
        std::cout << "主函数，第 " << i + 1 << " 次切换" << std::endl;
        source = source.resume();
    }

    return 0;
}
```

---

## 数据传递

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>

namespace ctx = boost::context;

struct transfer_data {
    int value;
};

int main() {
    transfer_data td{0};

    ctx::continuation c = ctx::callcc(
        [](ctx::continuation&& sink) {
            transfer_data* data = reinterpret_cast<transfer_data*>(
                sink.resume().data()
            );

            std::cout << "接收到数据: " << data->value << std::endl;

            data->value = 100;
            sink = sink.resume(data);

            std::cout << "再次接收: " << data->value << std::endl;

            return std::move(sink);
        }
    );

    td.value = 42;
    c = c.resume(&td);

    std::cout << "返回数据: " << td.value << std::endl;

    td.value = 200;
    c = c.resume(&td);

    return 0;
}
```

---

## 简单的协程实现

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>
#include <memory>

namespace ctx = boost::context;

class Generator {
public:
    Generator(std::function<void(std::function<void(int)>)> func) {
        context_ = ctx::callcc(
            [this, func](ctx::continuation&& sink) mutable {
                sink_ = std::move(sink);

                func([this](int value) {
                    current_value_ = value;
                    sink_ = sink_.resume();
                });

                has_next_ = false;
                return std::move(sink_);
            }
        );
    }

    bool has_next() const { return has_next_; }

    int next() {
        context_ = context_.resume();
        return current_value_;
    }

private:
    ctx::continuation context_;
    ctx::continuation sink_;
    int current_value_ = 0;
    bool has_next_ = true;
};

int main() {
    Generator gen([](std::function<void(int)> yield) {
        for (int i = 0; i < 5; ++i) {
            yield(i * i);
        }
    });

    while (gen.has_next()) {
        std::cout << gen.next() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 自定义栈

```cpp
#include <boost/context/continuation.hpp>
#include <boost/context/protected_fixedsize_stack.hpp>
#include <iostream>

namespace ctx = boost::context;

int main() {
    // 使用受保护的固定大小栈（64KB）
    ctx::protected_fixedsize_stack stack_allocator(64 * 1024);

    ctx::continuation c = ctx::callcc(
        std::allocator_arg,
        stack_allocator,
        [](ctx::continuation&& sink) {
            std::cout << "使用自定义栈大小的上下文" << std::endl;

            // 分配一些栈空间
            char buffer[1024];
            buffer[0] = 'A';

            std::cout << "缓冲区第一个字符: " << buffer[0] << std::endl;

            return std::move(sink);
        }
    );

    c = c.resume();

    return 0;
}
```

---

## 异常处理

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>
#include <exception>

namespace ctx = boost::context;

int main() {
    try {
        ctx::continuation c = ctx::callcc(
            [](ctx::continuation&& sink) {
                std::cout << "上下文开始" << std::endl;

                sink = sink.resume();

                std::cout << "抛出异常" << std::endl;
                throw std::runtime_error("上下文中的异常");

                return std::move(sink);
            }
        );

        std::cout << "第一次恢复" << std::endl;
        c = c.resume();

        std::cout << "第二次恢复" << std::endl;
        c = c.resume();

    } catch (const std::exception& e) {
        std::cout << "捕获异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 栈展开

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>

namespace ctx = boost::context;

class ResourceGuard {
public:
    ResourceGuard(const std::string& name) : name_(name) {
        std::cout << "ResourceGuard 构造: " << name_ << std::endl;
    }

    ~ResourceGuard() {
        std::cout << "ResourceGuard 析构: " << name_ << std::endl;
    }

private:
    std::string name_;
};

int main() {
    ctx::continuation c = ctx::callcc(
        [](ctx::continuation&& sink) {
            ResourceGuard guard("Context Resource");

            std::cout << "上下文中" << std::endl;
            sink = sink.resume();

            std::cout << "上下文恢复" << std::endl;

            return std::move(sink);
        }
    );

    std::cout << "主函数" << std::endl;
    c = c.resume();

    std::cout << "主函数结束" << std::endl;
    // c 离开作用域，上下文被销毁，ResourceGuard 析构

    return 0;
}
```

---

## 性能测试

```cpp
#include <boost/context/continuation.hpp>
#include <iostream>
#include <chrono>

namespace ctx = boost::context;

int main() {
    const int iterations = 1000000;
    int counter = 0;

    ctx::continuation c = ctx::callcc(
        [&counter, iterations](ctx::continuation&& sink) {
            for (int i = 0; i < iterations; ++i) {
                ++counter;
                sink = sink.resume();
            }
            return std::move(sink);
        }
    );

    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < iterations; ++i) {
        c = c.resume();
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

    std::cout << "执行 " << iterations << " 次上下文切换" << std::endl;
    std::cout << "总耗时: " << duration.count() << " 微秒" << std::endl;
    std::cout << "平均耗时: " << (double)duration.count() / iterations << " 微秒/次" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Context 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/context/doc/html/index.html)
- [上下文切换原理](https://www.boost.org/doc/libs/1_90_0/libs/context/doc/html/context/overview.html)
- [栈分配器](https://www.boost.org/doc/libs/1_90_0/libs/context/doc/html/context/stack.html)

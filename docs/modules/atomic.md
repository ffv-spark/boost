# Boost.Atomic - 原子操作库

## 概述

Boost.Atomic 提供原子操作和内存序，用于无锁编程。

**类型**: 仅头文件库

**注意**: C++11 已引入 `std::atomic`

---

## 快速开始

```cpp
#include <boost/atomic.hpp>
#include <boost/thread.hpp>
#include <iostream>

boost::atomic<int> counter(0);

void increment() {
    for (int i = 0; i < 10000; ++i) {
        ++counter;  // 原子递增
    }
}

int main() {
    boost::thread_group threads;

    for (int i = 0; i < 10; ++i) {
        threads.create_thread(increment);
    }

    threads.join_all();

    std::cout << "Counter: " << counter << std::endl;  // 应该是 100000

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_thread -lboost_system -lpthread -o example
```

---

## 原子操作

```cpp
#include <boost/atomic.hpp>
#include <iostream>

int main() {
    boost::atomic<int> value(10);

    // 读取
    int current = value.load();
    std::cout << "Current: " << current << std::endl;

    // 写入
    value.store(20);

    // 交换
    int old = value.exchange(30);
    std::cout << "Old: " << old << ", New: " << value << std::endl;

    // Compare and Swap
    int expected = 30;
    bool success = value.compare_exchange_strong(expected, 40);
    std::cout << "CAS success: " << success << std::endl;
    std::cout << "Value: " << value << std::endl;

    return 0;
}
```

---

## 内存序

```cpp
#include <boost/atomic.hpp>
#include <boost/thread.hpp>
#include <iostream>

boost::atomic<int> data(0);
boost::atomic<bool> ready(false);

void producer() {
    data.store(42, boost::memory_order_relaxed);
    ready.store(true, boost::memory_order_release);  // 释放语义
}

void consumer() {
    while (!ready.load(boost::memory_order_acquire)) {  // 获取语义
        // 自旋等待
    }
    std::cout << "Data: " << data.load(boost::memory_order_relaxed) << std::endl;
}

int main() {
    boost::thread t1(producer);
    boost::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

---

## 无锁栈（简化版）

```cpp
#include <boost/atomic.hpp>
#include <iostream>

template<typename T>
class LockFreeStack {
private:
    struct Node {
        T data;
        Node* next;

        Node(const T& val) : data(val), next(nullptr) {}
    };

    boost::atomic<Node*> head_;

public:
    LockFreeStack() : head_(nullptr) {}

    void push(const T& value) {
        Node* new_node = new Node(value);
        new_node->next = head_.load();

        while (!head_.compare_exchange_weak(new_node->next, new_node)) {
            // 重试
        }
    }

    bool pop(T& result) {
        Node* old_head = head_.load();

        while (old_head && !head_.compare_exchange_weak(old_head, old_head->next)) {
            // 重试
        }

        if (old_head) {
            result = old_head->data;
            delete old_head;
            return true;
        }

        return false;
    }
};

int main() {
    LockFreeStack<int> stack;

    stack.push(1);
    stack.push(2);
    stack.push(3);

    int value;
    while (stack.pop(value)) {
        std::cout << "Popped: " << value << std::endl;
    }

    return 0;
}
```

---

## 标志（Flag）

```cpp
#include <boost/atomic.hpp>
#include <boost/thread.hpp>
#include <iostream>

boost::atomic_flag lock = BOOST_ATOMIC_FLAG_INIT;

void critical_section(int id) {
    // 自旋锁
    while (lock.test_and_set(boost::memory_order_acquire)) {
        // 自旋等待
    }

    std::cout << "Thread " << id << " in critical section" << std::endl;
    boost::this_thread::sleep_for(boost::chrono::milliseconds(100));

    lock.clear(boost::memory_order_release);
}

int main() {
    boost::thread_group threads;

    for (int i = 0; i < 5; ++i) {
        threads.create_thread(boost::bind(critical_section, i));
    }

    threads.join_all();

    return 0;
}
```

---

## 最佳实践

1. **优先使用 std::atomic**: C++11 及以上
2. **理解内存序**: 正确使用内存序很重要
3. **避免 ABA 问题**: 使用版本号
4. **性能**: 原子操作比锁快，但仍有开销
5. **测试**: 无锁代码难以调试，需要充分测试

---

## 与 std::atomic 对比

```cpp
#include <boost/atomic.hpp>
#include <atomic>
#include <iostream>

int main() {
    // Boost.Atomic
    boost::atomic<int> b_atomic(0);
    ++b_atomic;

    // std::atomic (C++11)
    std::atomic<int> s_atomic(0);
    ++s_atomic;

    // API 基本相同
    std::cout << "Boost: " << b_atomic << std::endl;
    std::cout << "Std: " << s_atomic << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Atomic 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/atomic.html)
- [C++ 内存模型](https://en.cppreference.com/w/cpp/atomic/memory_order)

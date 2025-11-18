# Boost.Pool - 内存池库

## 概述

Boost.Pool 提供高效的内存分配器，通过预分配内存块来减少内存分配开销。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/pool/pool.hpp>
#include <iostream>

int main() {
    // 创建一个分配 int 大小内存的池
    boost::pool<> int_pool(sizeof(int));

    // 分配内存
    int* p1 = static_cast<int*>(int_pool.malloc());
    int* p2 = static_cast<int*>(int_pool.malloc());

    *p1 = 42;
    *p2 = 100;

    std::cout << "p1: " << *p1 << std::endl;
    std::cout << "p2: " << *p2 << std::endl;

    // 释放内存
    int_pool.free(p1);
    int_pool.free(p2);

    return 0;
}
```

---

## object_pool - 对象池

```cpp
#include <boost/pool/object_pool.hpp>
#include <iostream>
#include <string>

class Person {
public:
    Person(const std::string& name, int age)
        : name_(name), age_(age) {
        std::cout << "Person 构造: " << name_ << std::endl;
    }

    ~Person() {
        std::cout << "Person 析构: " << name_ << std::endl;
    }

    void introduce() const {
        std::cout << "我是 " << name_ << ", " << age_ << " 岁" << std::endl;
    }

private:
    std::string name_;
    int age_;
};

int main() {
    boost::object_pool<Person> person_pool;

    // 构造对象（调用构造函数）
    Person* p1 = person_pool.construct("Alice", 30);
    Person* p2 = person_pool.construct("Bob", 25);

    p1->introduce();
    p2->introduce();

    // 销毁对象（调用析构函数）
    person_pool.destroy(p1);
    person_pool.destroy(p2);

    return 0;
}
```

---

## singleton_pool - 单例池

```cpp
#include <boost/pool/singleton_pool.hpp>
#include <iostream>

// 定义单例池标签
struct MyPoolTag {};

typedef boost::singleton_pool<MyPoolTag, sizeof(int)> IntPool;

int main() {
    // 分配内存
    int* p1 = static_cast<int*>(IntPool::malloc());
    int* p2 = static_cast<int*>(IntPool::malloc());

    *p1 = 10;
    *p2 = 20;

    std::cout << "p1: " << *p1 << ", p2: " << *p2 << std::endl;

    // 释放内存
    IntPool::free(p1);
    IntPool::free(p2);

    // 释放所有内存
    IntPool::purge_memory();

    return 0;
}
```

---

## pool_allocator - STL 容器分配器

```cpp
#include <boost/pool/pool_alloc.hpp>
#include <iostream>
#include <vector>
#include <list>
#include <chrono>

int main() {
    const int size = 100000;

    // 使用 pool_allocator 的 vector
    {
        auto start = std::chrono::high_resolution_clock::now();

        std::vector<int, boost::pool_allocator<int>> vec;
        for (int i = 0; i < size; ++i) {
            vec.push_back(i);
        }

        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

        std::cout << "pool_allocator 耗时: " << duration.count() << " 微秒" << std::endl;
    }

    // 使用默认分配器的 vector
    {
        auto start = std::chrono::high_resolution_clock::now();

        std::vector<int> vec;
        for (int i = 0; i < size; ++i) {
            vec.push_back(i);
        }

        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

        std::cout << "默认 allocator 耗时: " << duration.count() << " 微秒" << std::endl;
    }

    return 0;
}
```

---

## fast_pool_allocator - 快速分配器

```cpp
#include <boost/pool/pool_alloc.hpp>
#include <iostream>
#include <list>

int main() {
    // 使用 fast_pool_allocator 的 list
    std::list<int, boost::fast_pool_allocator<int>> my_list;

    for (int i = 0; i < 10; ++i) {
        my_list.push_back(i * 10);
    }

    std::cout << "List 内容: ";
    for (int value : my_list) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    // 清理内存
    boost::singleton_pool<boost::fast_pool_allocator_tag,
                          sizeof(int)>::release_memory();

    return 0;
}
```

---

## 批量分配

```cpp
#include <boost/pool/pool.hpp>
#include <iostream>
#include <vector>

int main() {
    boost::pool<> my_pool(sizeof(int));

    const int count = 100;
    std::vector<int*> pointers;

    // 批量分配
    for (int i = 0; i < count; ++i) {
        int* p = static_cast<int*>(my_pool.malloc());
        if (p) {
            *p = i;
            pointers.push_back(p);
        }
    }

    std::cout << "分配了 " << pointers.size() << " 个对象" << std::endl;

    // 验证数据
    std::cout << "前 10 个值: ";
    for (int i = 0; i < 10 && i < pointers.size(); ++i) {
        std::cout << *pointers[i] << " ";
    }
    std::cout << std::endl;

    // 批量释放
    for (int* p : pointers) {
        my_pool.free(p);
    }

    return 0;
}
```

---

## 性能对比测试

```cpp
#include <boost/pool/object_pool.hpp>
#include <iostream>
#include <chrono>
#include <vector>

class TestObject {
public:
    TestObject(int value) : value_(value), data_{} {}
    int get_value() const { return value_; }

private:
    int value_;
    char data_[64];  // 增加对象大小
};

int main() {
    const int iterations = 10000;

    // 使用 object_pool
    {
        auto start = std::chrono::high_resolution_clock::now();

        boost::object_pool<TestObject> pool;
        std::vector<TestObject*> objects;

        for (int i = 0; i < iterations; ++i) {
            objects.push_back(pool.construct(i));
        }

        for (auto* obj : objects) {
            pool.destroy(obj);
        }

        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

        std::cout << "object_pool 耗时: " << duration.count() << " 微秒" << std::endl;
    }

    // 使用 new/delete
    {
        auto start = std::chrono::high_resolution_clock::now();

        std::vector<TestObject*> objects;

        for (int i = 0; i < iterations; ++i) {
            objects.push_back(new TestObject(i));
        }

        for (auto* obj : objects) {
            delete obj;
        }

        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

        std::cout << "new/delete 耗时: " << duration.count() << " 微秒" << std::endl;
    }

    return 0;
}
```

---

## 内存池统计

```cpp
#include <boost/pool/pool.hpp>
#include <iostream>

int main() {
    boost::pool<> my_pool(sizeof(int));

    // 分配一些内存
    std::vector<void*> blocks;
    for (int i = 0; i < 100; ++i) {
        blocks.push_back(my_pool.malloc());
    }

    // 获取统计信息
    std::cout << "已请求的块数: " << my_pool.get_requested_size() << std::endl;

    // 释放部分内存
    for (int i = 0; i < 50; ++i) {
        my_pool.free(blocks[i]);
    }

    // 检查是否可以释放系统内存
    if (my_pool.release_memory()) {
        std::cout << "成功释放未使用的系统内存" << std::endl;
    }

    // 清理剩余内存
    for (int i = 50; i < 100; ++i) {
        my_pool.free(blocks[i]);
    }

    return 0;
}
```

---

## 链表节点池

```cpp
#include <boost/pool/object_pool.hpp>
#include <iostream>
#include <string>

struct ListNode {
    int value;
    ListNode* next;

    ListNode(int v) : value(v), next(nullptr) {}
};

class LinkedList {
public:
    LinkedList() : head_(nullptr) {}

    ~LinkedList() {
        clear();
    }

    void push_front(int value) {
        ListNode* node = pool_.construct(value);
        node->next = head_;
        head_ = node;
    }

    void print() const {
        ListNode* current = head_;
        while (current) {
            std::cout << current->value << " -> ";
            current = current->next;
        }
        std::cout << "NULL" << std::endl;
    }

    void clear() {
        ListNode* current = head_;
        while (current) {
            ListNode* next = current->next;
            pool_.destroy(current);
            current = next;
        }
        head_ = nullptr;
    }

private:
    ListNode* head_;
    boost::object_pool<ListNode> pool_;
};

int main() {
    LinkedList list;

    for (int i = 0; i < 10; ++i) {
        list.push_front(i);
    }

    list.print();

    return 0;
}
```

---

## 参考资源

- [Boost.Pool 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/pool/doc/html/index.html)
- [内存池原理](https://www.boost.org/doc/libs/1_90_0/libs/pool/doc/html/boost_pool/pool/pooling.html)

# Boost.Smart Ptr - 智能指针库

## 概述

Boost.Smart Ptr 提供了多种智能指针类，用于自动管理动态分配的对象生命周期，防止内存泄漏。

**类型**: 仅头文件库（无需编译）

**主要组件**:
- `shared_ptr` - 共享所有权智能指针
- `weak_ptr` - 弱引用指针
- `scoped_ptr` - 独占所有权指针（已废弃，使用 `std::unique_ptr`）
- `intrusive_ptr` - 侵入式引用计数指针
- `make_shared` - 高效创建 shared_ptr

---

## 快速开始

```cpp
#include <boost/smart_ptr.hpp>
#include <iostream>

int main() {
    // 创建 shared_ptr
    boost::shared_ptr<int> ptr = boost::make_shared<int>(42);
    std::cout << "值: " << *ptr << std::endl;
    std::cout << "引用计数: " << ptr.use_count() << std::endl;

    return 0;
}
```

---

## shared_ptr - 共享所有权指针

### 基本用法

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/make_shared.hpp>
#include <iostream>
#include <vector>

class MyClass {
public:
    MyClass(int val) : value(val) {
        std::cout << "构造 MyClass(" << value << ")" << std::endl;
    }

    ~MyClass() {
        std::cout << "析构 MyClass(" << value << ")" << std::endl;
    }

    void print() const {
        std::cout << "MyClass::value = " << value << std::endl;
    }

private:
    int value;
};

int main() {
    // 1. 创建 shared_ptr
    boost::shared_ptr<MyClass> ptr1 = boost::make_shared<MyClass>(100);

    // 2. 共享所有权
    boost::shared_ptr<MyClass> ptr2 = ptr1;

    std::cout << "引用计数: " << ptr1.use_count() << std::endl; // 输出: 2

    // 3. 使用指针
    ptr1->print();
    (*ptr2).print();

    // 4. 在容器中使用
    std::vector<boost::shared_ptr<MyClass>> vec;
    vec.push_back(boost::make_shared<MyClass>(1));
    vec.push_back(boost::make_shared<MyClass>(2));
    vec.push_back(boost::make_shared<MyClass>(3));

    // 遍历
    for (const auto& p : vec) {
        p->print();
    }

    // 所有 shared_ptr 超出作用域时，对象自动删除
    return 0;
}
```

### 自定义删除器

```cpp
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <cstdio>

// 自定义删除器 - 用于文件句柄
void close_file(FILE* fp) {
    if (fp) {
        std::cout << "关闭文件" << std::endl;
        fclose(fp);
    }
}

int main() {
    // 使用自定义删除器
    boost::shared_ptr<FILE> file(
        fopen("test.txt", "w"),
        close_file
    );

    if (file) {
        fprintf(file.get(), "Hello, Boost!\n");
    }

    // 文件自动关闭
    return 0;
}
```

### 数组支持

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/shared_array.hpp>
#include <iostream>

int main() {
    // shared_array - 用于数组
    boost::shared_array<int> arr(new int[5]);

    for (int i = 0; i < 5; ++i) {
        arr[i] = i * 10;
    }

    for (int i = 0; i < 5; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    // 数组自动删除（使用 delete[]）
    return 0;
}
```

---

## weak_ptr - 弱引用指针

用于解决循环引用问题。

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>
#include <iostream>

class Node {
public:
    std::string name;
    boost::shared_ptr<Node> next;
    boost::weak_ptr<Node> prev; // 使用 weak_ptr 避免循环引用

    Node(const std::string& n) : name(n) {
        std::cout << "创建节点: " << name << std::endl;
    }

    ~Node() {
        std::cout << "删除节点: " << name << std::endl;
    }
};

int main() {
    boost::shared_ptr<Node> node1 = boost::make_shared<Node>("Node1");
    boost::shared_ptr<Node> node2 = boost::make_shared<Node>("Node2");

    // 创建双向链表
    node1->next = node2;
    node2->prev = node1; // weak_ptr 不增加引用计数

    std::cout << "node1 引用计数: " << node1.use_count() << std::endl; // 1
    std::cout << "node2 引用计数: " << node2.use_count() << std::endl; // 2

    // 使用 weak_ptr
    if (boost::shared_ptr<Node> prev = node2->prev.lock()) {
        std::cout << "node2 的前驱: " << prev->name << std::endl;
    }

    // 正确释放内存，无循环引用
    return 0;
}
```

---

## intrusive_ptr - 侵入式引用计数

适用于已有引用计数的类。

```cpp
#include <boost/intrusive_ptr.hpp>
#include <boost/atomic.hpp>
#include <iostream>

class RefCounted {
private:
    mutable boost::atomic<int> ref_count_;

public:
    RefCounted() : ref_count_(0) {
        std::cout << "构造 RefCounted" << std::endl;
    }

    virtual ~RefCounted() {
        std::cout << "析构 RefCounted" << std::endl;
    }

    friend void intrusive_ptr_add_ref(const RefCounted* p) {
        ++p->ref_count_;
    }

    friend void intrusive_ptr_release(const RefCounted* p) {
        if (--p->ref_count_ == 0) {
            delete p;
        }
    }

    int use_count() const {
        return ref_count_;
    }
};

int main() {
    boost::intrusive_ptr<RefCounted> ptr1(new RefCounted());
    std::cout << "引用计数: " << ptr1->use_count() << std::endl;

    {
        boost::intrusive_ptr<RefCounted> ptr2 = ptr1;
        std::cout << "引用计数: " << ptr1->use_count() << std::endl;
    }

    std::cout << "引用计数: " << ptr1->use_count() << std::endl;

    return 0;
}
```

---

## 性能优化技巧

### 使用 make_shared

```cpp
#include <boost/make_shared.hpp>
#include <boost/shared_ptr.hpp>

class Data {
    int values[1000];
public:
    Data() {}
};

int main() {
    // 推荐：一次内存分配
    auto ptr1 = boost::make_shared<Data>();

    // 不推荐：两次内存分配（对象 + 控制块）
    boost::shared_ptr<Data> ptr2(new Data());

    return 0;
}
```

### 避免循环引用

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>

class Parent;
class Child;

class Parent {
public:
    boost::shared_ptr<Child> child;
    ~Parent() { std::cout << "~Parent()" << std::endl; }
};

class Child {
public:
    boost::weak_ptr<Parent> parent; // 使用 weak_ptr 打破循环
    ~Child() { std::cout << "~Child()" << std::endl; }
};

int main() {
    auto parent = boost::make_shared<Parent>();
    auto child = boost::make_shared<Child>();

    parent->child = child;
    child->parent = parent;

    // 正确释放内存
    return 0;
}
```

---

## 线程安全

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/make_shared.hpp>
#include <boost/thread.hpp>
#include <iostream>

boost::shared_ptr<int> global_ptr;

void thread_func(int id) {
    // shared_ptr 的拷贝和赋值是线程安全的
    boost::shared_ptr<int> local = global_ptr;

    std::cout << "线程 " << id << ": " << *local << std::endl;
}

int main() {
    global_ptr = boost::make_shared<int>(42);

    boost::thread t1(thread_func, 1);
    boost::thread t2(thread_func, 2);
    boost::thread t3(thread_func, 3);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

---

## 与 std::shared_ptr 的比较

| 特性 | boost::shared_ptr | std::shared_ptr |
|------|-------------------|-----------------|
| C++ 标准 | C++98 及以上 | C++11 及以上 |
| 性能 | 略慢 | 略快 |
| 原子操作 | 可选 | 强制 |
| make_shared | 支持 | 支持 |
| 自定义分配器 | 有限支持 | 完全支持 |

**建议**: 如果使用 C++11 或更高版本，优先使用 `std::shared_ptr`。

---

## 常见用例

### 工厂模式

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/make_shared.hpp>

class Product {
public:
    virtual ~Product() {}
    virtual void use() = 0;
};

class ConcreteProduct : public Product {
public:
    void use() override {
        std::cout << "使用具体产品" << std::endl;
    }
};

boost::shared_ptr<Product> createProduct() {
    return boost::make_shared<ConcreteProduct>();
}

int main() {
    auto product = createProduct();
    product->use();
    return 0;
}
```

### PIMPL 模式

```cpp
// widget.h
#include <boost/shared_ptr.hpp>

class Widget {
public:
    Widget();
    ~Widget();
    void doSomething();

private:
    class Impl;
    boost::shared_ptr<Impl> pimpl_;
};

// widget.cpp
class Widget::Impl {
public:
    void doSomething() {
        // 实现细节
    }
};

Widget::Widget() : pimpl_(boost::make_shared<Impl>()) {}
Widget::~Widget() = default;

void Widget::doSomething() {
    pimpl_->doSomething();
}
```

---

## 编译和运行

```bash
# 仅头文件库，无需链接
g++ -std=c++11 -I/path/to/boost example.cpp -o example

# 如果使用 thread 示例，需要链接 boost_thread
g++ -std=c++11 -I/path/to/boost thread_example.cpp \
    -lboost_thread -lpthread -o thread_example
```

---

## 最佳实践

1. **优先使用 make_shared**: 减少内存分配次数
2. **避免循环引用**: 使用 weak_ptr 打破循环
3. **不要混用**: 不要混用裸指针和智能指针管理同一对象
4. **线程安全**: 智能指针本身的引用计数是线程安全的，但指向的对象不是
5. **性能考虑**: 对于性能关键代码，考虑使用 intrusive_ptr

---

## 参考资源

- [Boost.Smart Ptr 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/smart_ptr/smart_ptr.htm)
- [C++ 智能指针最佳实践](https://www.boost.org/doc/libs/1_90_0/libs/smart_ptr/sp_techniques.html)

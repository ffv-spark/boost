# Boost.SmartPtr - 智能指针库

## 概述

Boost.SmartPtr 提供多种智能指针，用于自动内存管理。

**类型**: 仅头文件库

**注意**: C++11 引入了 `std::shared_ptr`、`std::unique_ptr` 等，建议使用标准库版本

---

## shared_ptr - 共享所有权

```cpp
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <string>

class Person {
public:
    Person(const std::string& name) : name_(name) {
        std::cout << "Person 构造: " << name_ << std::endl;
    }

    ~Person() {
        std::cout << "Person 析构: " << name_ << std::endl;
    }

    void greet() const {
        std::cout << "你好，我是 " << name_ << std::endl;
    }

private:
    std::string name_;
};

int main() {
    boost::shared_ptr<Person> p1(new Person("Alice"));
    std::cout << "引用计数: " << p1.use_count() << std::endl;

    {
        boost::shared_ptr<Person> p2 = p1;
        std::cout << "引用计数: " << p1.use_count() << std::endl;

        p2->greet();
    }

    std::cout << "引用计数: " << p1.use_count() << std::endl;

    return 0;
}
// 离开作用域时，Person 自动析构
```

---

## make_shared - 高效创建

```cpp
#include <boost/make_shared.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>

class Data {
public:
    Data(int value) : value_(value) {
        std::cout << "Data 构造: " << value_ << std::endl;
    }

    ~Data() {
        std::cout << "Data 析构: " << value_ << std::endl;
    }

    int get_value() const { return value_; }

private:
    int value_;
};

int main() {
    // 使用 make_shared 更高效（一次内存分配）
    auto ptr = boost::make_shared<Data>(42);

    std::cout << "值: " << ptr->get_value() << std::endl;
    std::cout << "引用计数: " << ptr.use_count() << std::endl;

    return 0;
}
```

---

## weak_ptr - 弱引用

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>
#include <iostream>

int main() {
    boost::weak_ptr<int> weak;

    {
        auto shared = boost::make_shared<int>(100);
        weak = shared;

        std::cout << "shared 引用计数: " << shared.use_count() << std::endl;
        std::cout << "weak 过期: " << (weak.expired() ? "是" : "否") << std::endl;

        // 从 weak_ptr 获取 shared_ptr
        if (auto locked = weak.lock()) {
            std::cout << "值: " << *locked << std::endl;
        }
    }

    // shared_ptr 已销毁
    std::cout << "weak 过期: " << (weak.expired() ? "是" : "否") << std::endl;

    return 0;
}
```

---

## 解决循环引用

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>
#include <boost/enable_shared_from_this.hpp>
#include <iostream>
#include <string>

class Node {
public:
    std::string name;
    boost::shared_ptr<Node> next;
    boost::weak_ptr<Node> prev;  // 使用 weak_ptr 避免循环引用

    Node(const std::string& n) : name(n) {
        std::cout << "Node 构造: " << name << std::endl;
    }

    ~Node() {
        std::cout << "Node 析构: " << name << std::endl;
    }
};

int main() {
    auto node1 = boost::make_shared<Node>("Node1");
    auto node2 = boost::make_shared<Node>("Node2");

    node1->next = node2;
    node2->prev = node1;  // weak_ptr 不增加引用计数

    std::cout << "node1 引用计数: " << node1.use_count() << std::endl;
    std::cout << "node2 引用计数: " << node2.use_count() << std::endl;

    return 0;
}
// 正确析构，无内存泄漏
```

---

## enable_shared_from_this

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/enable_shared_from_this.hpp>
#include <boost/make_shared.hpp>
#include <iostream>

class Widget : public boost::enable_shared_from_this<Widget> {
public:
    void register_callback() {
        // 获取指向自己的 shared_ptr
        auto self = shared_from_this();
        std::cout << "注册回调，引用计数: " << self.use_count() << std::endl;
    }

    void process() {
        std::cout << "处理中..." << std::endl;
    }
};

int main() {
    auto widget = boost::make_shared<Widget>();
    std::cout << "初始引用计数: " << widget.use_count() << std::endl;

    widget->register_callback();
    std::cout << "调用后引用计数: " << widget.use_count() << std::endl;

    return 0;
}
```

---

## scoped_ptr - 独占所有权

```cpp
#include <boost/scoped_ptr.hpp>
#include <iostream>

class Resource {
public:
    Resource() {
        std::cout << "Resource 构造" << std::endl;
    }

    ~Resource() {
        std::cout << "Resource 析构" << std::endl;
    }

    void use() {
        std::cout << "使用资源" << std::endl;
    }
};

int main() {
    boost::scoped_ptr<Resource> ptr(new Resource());

    ptr->use();

    // scoped_ptr 不能复制或赋值
    // boost::scoped_ptr<Resource> ptr2 = ptr;  // 错误！

    return 0;
}
// 离开作用域自动删除
```

---

## scoped_array - 数组管理

```cpp
#include <boost/scoped_array.hpp>
#include <iostream>

int main() {
    const int size = 10;
    boost::scoped_array<int> arr(new int[size]);

    // 初始化数组
    for (int i = 0; i < size; ++i) {
        arr[i] = i * i;
    }

    // 访问数组
    std::cout << "数组元素: ";
    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    // 获取原始指针
    int* raw_ptr = arr.get();
    std::cout << "第一个元素: " << raw_ptr[0] << std::endl;

    return 0;
}
// 自动删除数组
```

---

## shared_array - 共享数组

```cpp
#include <boost/shared_array.hpp>
#include <iostream>

int main() {
    const int size = 5;
    boost::shared_array<int> arr1(new int[size]);

    for (int i = 0; i < size; ++i) {
        arr1[i] = i + 1;
    }

    // 共享所有权
    boost::shared_array<int> arr2 = arr1;

    std::cout << "arr1: ";
    for (int i = 0; i < size; ++i) {
        std::cout << arr1[i] << " ";
    }
    std::cout << std::endl;

    std::cout << "arr2: ";
    for (int i = 0; i < size; ++i) {
        std::cout << arr2[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## intrusive_ptr - 侵入式智能指针

```cpp
#include <boost/intrusive_ptr.hpp>
#include <iostream>

class RefCounted {
public:
    RefCounted() : ref_count_(0) {}

    int get_ref_count() const { return ref_count_; }

    friend void intrusive_ptr_add_ref(RefCounted* p) {
        ++p->ref_count_;
    }

    friend void intrusive_ptr_release(RefCounted* p) {
        if (--p->ref_count_ == 0) {
            delete p;
        }
    }

protected:
    virtual ~RefCounted() {
        std::cout << "RefCounted 析构" << std::endl;
    }

private:
    mutable int ref_count_;
};

class MyObject : public RefCounted {
public:
    MyObject(int value) : value_(value) {
        std::cout << "MyObject 构造: " << value_ << std::endl;
    }

    int get_value() const { return value_; }

protected:
    ~MyObject() {
        std::cout << "MyObject 析构: " << value_ << std::endl;
    }

private:
    int value_;
};

int main() {
    boost::intrusive_ptr<MyObject> ptr1(new MyObject(42));
    std::cout << "引用计数: " << ptr1->get_ref_count() << std::endl;

    {
        boost::intrusive_ptr<MyObject> ptr2 = ptr1;
        std::cout << "引用计数: " << ptr1->get_ref_count() << std::endl;
    }

    std::cout << "引用计数: " << ptr1->get_ref_count() << std::endl;

    return 0;
}
```

---

## 自定义删除器

```cpp
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <cstdio>

void file_closer(FILE* fp) {
    if (fp) {
        std::cout << "关闭文件" << std::endl;
        std::fclose(fp);
    }
}

int main() {
    // 使用自定义删除器管理 FILE*
    boost::shared_ptr<FILE> file(
        std::fopen("test.txt", "w"),
        file_closer
    );

    if (file) {
        std::fprintf(file.get(), "Hello, Boost.SmartPtr!\n");
    }

    return 0;
}
// 文件自动关闭
```

---

## 与标准库对比

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/make_shared.hpp>
#include <memory>
#include <iostream>

int main() {
    // Boost 版本
    auto boost_ptr = boost::make_shared<int>(42);
    std::cout << "Boost: " << *boost_ptr << std::endl;

    // 标准库版本 (C++11)
    auto std_ptr = std::make_shared<int>(42);
    std::cout << "std: " << *std_ptr << std::endl;

    // 用法基本相同
    std::cout << "Boost 引用计数: " << boost_ptr.use_count() << std::endl;
    std::cout << "std 引用计数: " << std_ptr.use_count() << std::endl;

    return 0;
}
```

**建议**: 在 C++11 及以后的项目中，优先使用标准库的智能指针。

---

## 参考资源

- [Boost.SmartPtr 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/smart_ptr/doc/html/smart_ptr.html)
- [C++11 智能指针](https://en.cppreference.com/w/cpp/memory)

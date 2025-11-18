# Boost.Variant - 变体类型库

## 概述

Boost.Variant 提供类型安全的联合体（union），可以存储多种类型中的一种。

**类型**: 仅头文件库

**注意**: C++17 引入了 std::variant，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    // 定义可以存储 int、double 或 string 的变体
    boost::variant<int, double, std::string> v;

    v = 42;
    std::cout << "整数: " << boost::get<int>(v) << std::endl;

    v = 3.14;
    std::cout << "浮点: " << boost::get<double>(v) << std::endl;

    v = "hello";
    std::cout << "字符串: " << boost::get<std::string>(v) << std::endl;

    return 0;
}
```

---

## 访问值

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    boost::variant<int, double, std::string> v = 42;

    // 使用 get 访问（必须指定正确类型）
    try {
        std::cout << "int: " << boost::get<int>(v) << std::endl;

        // 尝试获取错误类型会抛出异常
        // std::cout << boost::get<double>(v) << std::endl;  // 抛出 bad_get
    } catch (const boost::bad_get& e) {
        std::cout << "错误: " << e.what() << std::endl;
    }

    // 使用指针版本（返回 nullptr 表示类型不匹配）
    int* pi = boost::get<int>(&v);
    if (pi) {
        std::cout << "通过指针获取: " << *pi << std::endl;
    }

    double* pd = boost::get<double>(&v);
    if (!pd) {
        std::cout << "不是 double 类型" << std::endl;
    }

    return 0;
}
```

---

## which 函数

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    boost::variant<int, double, std::string> v;

    v = 42;
    std::cout << "当前类型索引: " << v.which() << std::endl;  // 0

    v = 3.14;
    std::cout << "当前类型索引: " << v.which() << std::endl;  // 1

    v = "hello";
    std::cout << "当前类型索引: " << v.which() << std::endl;  // 2

    return 0;
}
```

---

## 访问器模式

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 定义访问器
struct print_visitor : public boost::static_visitor<void> {
    void operator()(int i) const {
        std::cout << "整数: " << i << std::endl;
    }

    void operator()(double d) const {
        std::cout << "浮点: " << d << std::endl;
    }

    void operator()(const std::string& s) const {
        std::cout << "字符串: " << s << std::endl;
    }
};

int main() {
    boost::variant<int, double, std::string> v1 = 42;
    boost::variant<int, double, std::string> v2 = 3.14;
    boost::variant<int, double, std::string> v3 = "hello";

    print_visitor visitor;

    boost::apply_visitor(visitor, v1);
    boost::apply_visitor(visitor, v2);
    boost::apply_visitor(visitor, v3);

    return 0;
}
```

---

## 返回值的访问器

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 计算"长度"的访问器
struct length_visitor : public boost::static_visitor<size_t> {
    size_t operator()(int i) const {
        return 0;  // 整数没有长度
    }

    size_t operator()(double d) const {
        return 0;  // 浮点数没有长度
    }

    size_t operator()(const std::string& s) const {
        return s.length();  // 字符串有长度
    }
};

int main() {
    boost::variant<int, double, std::string> v1 = "hello world";
    boost::variant<int, double, std::string> v2 = 42;

    length_visitor visitor;

    size_t len1 = boost::apply_visitor(visitor, v1);
    size_t len2 = boost::apply_visitor(visitor, v2);

    std::cout << "v1 长度: " << len1 << std::endl;
    std::cout << "v2 长度: " << len2 << std::endl;

    return 0;
}
```

---

## 递归变体

```cpp
#include <boost/variant.hpp>
#include <boost/variant/recursive_wrapper.hpp>
#include <iostream>
#include <vector>

// 递归数据结构：树节点
struct tree_node;

typedef boost::variant<
    int,
    boost::recursive_wrapper<std::vector<tree_node>>
> tree_node;

// 打印树的访问器
struct print_tree : public boost::static_visitor<void> {
    void operator()(int value) const {
        std::cout << value << " ";
    }

    void operator()(const std::vector<tree_node>& children) const {
        std::cout << "[ ";
        for (const auto& child : children) {
            boost::apply_visitor(*this, child);
        }
        std::cout << "] ";
    }
};

int main() {
    // 构建树: [ 1 [ 2 3 ] 4 ]
    tree_node tree = std::vector<tree_node>{
        tree_node(1),
        tree_node(std::vector<tree_node>{
            tree_node(2),
            tree_node(3)
        }),
        tree_node(4)
    };

    print_tree visitor;
    boost::apply_visitor(visitor, tree);
    std::cout << std::endl;

    return 0;
}
```

---

## 容器中的变体

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    using variant_t = boost::variant<int, double, std::string>;

    std::vector<variant_t> mixed_data;
    mixed_data.push_back(42);
    mixed_data.push_back(3.14);
    mixed_data.push_back("hello");
    mixed_data.push_back(100);
    mixed_data.push_back("world");

    std::cout << "混合数据:\n";
    for (const auto& item : mixed_data) {
        if (const int* pi = boost::get<int>(&item)) {
            std::cout << "  int: " << *pi << std::endl;
        } else if (const double* pd = boost::get<double>(&item)) {
            std::cout << "  double: " << *pd << std::endl;
        } else if (const std::string* ps = boost::get<std::string>(&item)) {
            std::cout << "  string: " << *ps << std::endl;
        }
    }

    return 0;
}
```

---

## 二元访问器

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 相等性检查访问器
struct equality_visitor : public boost::static_visitor<bool> {
    template<typename T, typename U>
    bool operator()(const T&, const U&) const {
        return false;  // 不同类型总是不相等
    }

    template<typename T>
    bool operator()(const T& lhs, const T& rhs) const {
        return lhs == rhs;  // 相同类型比较值
    }
};

int main() {
    boost::variant<int, double, std::string> v1 = 42;
    boost::variant<int, double, std::string> v2 = 42;
    boost::variant<int, double, std::string> v3 = 3.14;

    equality_visitor visitor;

    std::cout << std::boolalpha;
    std::cout << "v1 == v2: " << boost::apply_visitor(visitor, v1, v2) << std::endl;
    std::cout << "v1 == v3: " << boost::apply_visitor(visitor, v1, v3) << std::endl;

    return 0;
}
```

---

## 类型安全性

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 与 union 对比
union unsafe_union {
    int i;
    double d;
    // char* s;  // 不能包含非 POD 类型
};

int main() {
    // C union（不安全）
    unsafe_union u;
    u.i = 42;
    // 不知道当前存储的是什么类型
    // std::cout << u.d << std::endl;  // 未定义行为！

    // Boost.Variant（类型安全）
    boost::variant<int, double, std::string> v = 42;

    try {
        // 类型检查会在运行时进行
        double d = boost::get<double>(v);  // 抛出异常
    } catch (const boost::bad_get& e) {
        std::cout << "类型不匹配，捕获异常" << std::endl;
    }

    // 正确的访问
    std::cout << "值: " << boost::get<int>(v) << std::endl;

    return 0;
}
```

---

## 默认构造

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    // 默认构造使用第一个类型的默认值
    boost::variant<int, double, std::string> v1;
    std::cout << "v1 (默认): " << boost::get<int>(v1) << std::endl;  // 0

    boost::variant<std::string, int, double> v2;
    std::cout << "v2 (默认): [" << boost::get<std::string>(v2) << "]" << std::endl;  // ""

    return 0;
}
```

---

## 参考资源

- [Boost.Variant 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/variant.html)
- [std::variant 参考](https://en.cppreference.com/w/cpp/utility/variant)

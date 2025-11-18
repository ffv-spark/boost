# Boost.Any - 任意类型容器

## 概述

Boost.Any 提供类型安全的任意类型值容器，可以存储任何可复制构造的类型。

**类型**: 仅头文件库

**注意**: C++17 引入了 std::any，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

int main() {
    boost::any a;

    // 存储整数
    a = 42;
    std::cout << "整数: " << boost::any_cast<int>(a) << std::endl;

    // 存储字符串
    a = std::string("hello");
    std::cout << "字符串: " << boost::any_cast<std::string>(a) << std::endl;

    // 存储浮点数
    a = 3.14;
    std::cout << "浮点数: " << boost::any_cast<double>(a) << std::endl;

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

int main() {
    boost::any a;

    // 检查是否为空
    std::cout << std::boolalpha;
    std::cout << "初始为空: " << a.empty() << std::endl;

    // 赋值
    a = 100;
    std::cout << "赋值后为空: " << a.empty() << std::endl;

    // 获取类型信息
    std::cout << "类型: " << a.type().name() << std::endl;

    // 清空
    a.clear();
    std::cout << "清空后为空: " << a.empty() << std::endl;

    return 0;
}
```

---

## any_cast 使用

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

int main() {
    boost::any a = 42;

    try {
        // 正确的类型转换
        int i = boost::any_cast<int>(a);
        std::cout << "整数: " << i << std::endl;

        // 错误的类型转换（抛出异常）
        std::string s = boost::any_cast<std::string>(a);
    } catch (const boost::bad_any_cast& e) {
        std::cout << "类型转换失败: " << e.what() << std::endl;
    }

    // 使用指针版本（不抛异常）
    int* pi = boost::any_cast<int>(&a);
    if (pi) {
        std::cout << "通过指针获取: " << *pi << std::endl;
    }

    std::string* ps = boost::any_cast<std::string>(&a);
    if (!ps) {
        std::cout << "不是 string 类型" << std::endl;
    }

    return 0;
}
```

---

## 容器中的 any

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<boost::any> mixed_data;

    // 存储不同类型
    mixed_data.push_back(42);
    mixed_data.push_back(std::string("hello"));
    mixed_data.push_back(3.14);
    mixed_data.push_back(true);

    std::cout << "混合数据:\n";
    for (const auto& item : mixed_data) {
        if (item.type() == typeid(int)) {
            std::cout << "  int: " << boost::any_cast<int>(item) << std::endl;
        } else if (item.type() == typeid(std::string)) {
            std::cout << "  string: " << boost::any_cast<std::string>(item) << std::endl;
        } else if (item.type() == typeid(double)) {
            std::cout << "  double: " << boost::any_cast<double>(item) << std::endl;
        } else if (item.type() == typeid(bool)) {
            std::cout << "  bool: " << std::boolalpha << boost::any_cast<bool>(item) << std::endl;
        }
    }

    return 0;
}
```

---

## 自定义类型

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;

    Person(std::string n, int a) : name(n), age(a) {}
};

int main() {
    boost::any a = Person("Alice", 30);

    try {
        Person p = boost::any_cast<Person>(a);
        std::cout << "姓名: " << p.name << std::endl;
        std::cout << "年龄: " << p.age << std::endl;
    } catch (const boost::bad_any_cast& e) {
        std::cout << "转换失败" << std::endl;
    }

    return 0;
}
```

---

## 类型判断

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>
#include <typeinfo>

void print_any(const boost::any& a) {
    if (a.empty()) {
        std::cout << "空值" << std::endl;
    } else if (a.type() == typeid(int)) {
        std::cout << "整数: " << boost::any_cast<int>(a) << std::endl;
    } else if (a.type() == typeid(double)) {
        std::cout << "浮点数: " << boost::any_cast<double>(a) << std::endl;
    } else if (a.type() == typeid(std::string)) {
        std::cout << "字符串: " << boost::any_cast<std::string>(a) << std::endl;
    } else {
        std::cout << "未知类型" << std::endl;
    }
}

int main() {
    print_any(boost::any());
    print_any(boost::any(42));
    print_any(boost::any(3.14));
    print_any(boost::any(std::string("hello")));

    return 0;
}
```

---

## map 中使用 any

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, boost::any> config;

    // 存储配置
    config["server.host"] = std::string("localhost");
    config["server.port"] = 8080;
    config["server.timeout"] = 30.5;
    config["server.enabled"] = true;

    // 读取配置
    std::string host = boost::any_cast<std::string>(config["server.host"]);
    int port = boost::any_cast<int>(config["server.port"]);
    double timeout = boost::any_cast<double>(config["server.timeout"]);
    bool enabled = boost::any_cast<bool>(config["server.enabled"]);

    std::cout << "服务器配置:\n";
    std::cout << "  主机: " << host << std::endl;
    std::cout << "  端口: " << port << std::endl;
    std::cout << "  超时: " << timeout << "秒" << std::endl;
    std::cout << "  启用: " << std::boolalpha << enabled << std::endl;

    return 0;
}
```

---

## 函数返回任意类型

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

boost::any get_value(const std::string& key) {
    if (key == "age") {
        return 30;
    } else if (key == "name") {
        return std::string("Alice");
    } else if (key == "salary") {
        return 50000.0;
    }
    return boost::any();
}

int main() {
    auto age = get_value("age");
    auto name = get_value("name");
    auto salary = get_value("salary");
    auto unknown = get_value("unknown");

    if (!age.empty()) {
        std::cout << "年龄: " << boost::any_cast<int>(age) << std::endl;
    }

    if (!name.empty()) {
        std::cout << "姓名: " << boost::any_cast<std::string>(name) << std::endl;
    }

    if (!salary.empty()) {
        std::cout << "薪水: " << boost::any_cast<double>(salary) << std::endl;
    }

    if (unknown.empty()) {
        std::cout << "未知键返回空值" << std::endl;
    }

    return 0;
}
```

---

## 与 variant 对比

```cpp
#include <boost/any.hpp>
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    // variant 需要预先指定类型列表
    boost::variant<int, double, std::string> v;
    v = 42;
    v = 3.14;
    v = std::string("hello");

    // any 可以存储任何类型
    boost::any a;
    a = 42;
    a = 3.14;
    a = std::string("hello");

    // variant 更高效，any 更灵活
    std::cout << "variant 存储类型有限，性能更好" << std::endl;
    std::cout << "any 可存储任意类型，更灵活" << std::endl;

    return 0;
}
```

---

## 异常安全

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

int main() {
    boost::any a = 42;

    // 安全的转换方式
    if (int* pi = boost::any_cast<int>(&a)) {
        std::cout << "安全转换: " << *pi << std::endl;
    }

    // 修改值
    if (int* pi = boost::any_cast<int>(&a)) {
        *pi = 100;
    }

    std::cout << "修改后: " << boost::any_cast<int>(a) << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Any 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/any.html)
- [std::any 参考](https://en.cppreference.com/w/cpp/utility/any)

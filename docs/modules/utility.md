# Boost.Utility - 实用工具集合

## 概述

Boost.Utility 提供一系列通用的实用工具类和函数，包括 noncopyable、addressof 等。

**类型**: 仅头文件库

---

## noncopyable - 禁止拷贝

```cpp
#include <boost/noncopyable.hpp>
#include <iostream>

class Resource : private boost::noncopyable {
public:
    Resource(int id) : id_(id) {
        std::cout << "Resource " << id_ << " 创建" << std::endl;
    }

    ~Resource() {
        std::cout << "Resource " << id_ << " 销毁" << std::endl;
    }

    int get_id() const { return id_; }

private:
    int id_;
};

int main() {
    Resource r1(1);

    // 以下代码会编译错误：
    // Resource r2 = r1;  // 错误：拷贝构造被删除
    // Resource r3(2);
    // r3 = r1;           // 错误：拷贝赋值被删除

    std::cout << "Resource ID: " << r1.get_id() << std::endl;

    return 0;
}
```

---

## addressof - 获取真实地址

```cpp
#include <boost/core/addressof.hpp>
#include <iostream>

class Tricky {
public:
    // 重载了 operator&
    Tricky* operator&() {
        std::cout << "operator& 被调用" << std::endl;
        return nullptr;
    }

    void display() const {
        std::cout << "Tricky 对象" << std::endl;
    }
};

int main() {
    Tricky obj;

    // 使用 & 运算符（调用重载版本）
    Tricky* p1 = &obj;
    std::cout << "& 运算符返回: " << p1 << std::endl;

    // 使用 addressof 获取真实地址
    Tricky* p2 = boost::addressof(obj);
    std::cout << "addressof 返回: " << p2 << std::endl;

    if (p2) {
        p2->display();
    }

    return 0;
}
```

---

## checked_delete - 安全删除

```cpp
#include <boost/core/checked_delete.hpp>
#include <iostream>

class Incomplete;  // 前向声明

class Complete {
public:
    Complete() {
        std::cout << "Complete 构造" << std::endl;
    }

    ~Complete() {
        std::cout << "Complete 析构" << std::endl;
    }
};

int main() {
    Complete* p = new Complete();

    // 安全删除：确保类型完整
    boost::checked_delete(p);

    // 如果对不完整类型使用 checked_delete，会产生编译错误
    // Incomplete* p2 = ...;
    // boost::checked_delete(p2);  // 编译错误

    return 0;
}
```

---

## enable_if - 条件编译

```cpp
#include <boost/utility/enable_if.hpp>
#include <boost/type_traits.hpp>
#include <iostream>

// 只为整数类型启用
template<typename T>
typename boost::enable_if<boost::is_integral<T>, void>::type
print_value(T value) {
    std::cout << "整数: " << value << std::endl;
}

// 只为浮点类型启用
template<typename T>
typename boost::enable_if<boost::is_floating_point<T>, void>::type
print_value(T value) {
    std::cout << "浮点数: " << value << std::endl;
}

int main() {
    print_value(42);      // 调用整数版本
    print_value(3.14);    // 调用浮点数版本

    // print_value("hello");  // 编译错误：没有匹配的版本

    return 0;
}
```

---

## swap - 高效交换

```cpp
#include <boost/core/swap.hpp>
#include <iostream>
#include <vector>

class MyClass {
public:
    MyClass(int size) : data_(size, 0) {
        std::cout << "构造，大小: " << size << std::endl;
    }

    friend void swap(MyClass& a, MyClass& b) {
        std::cout << "使用自定义 swap" << std::endl;
        a.data_.swap(b.data_);
    }

    size_t size() const { return data_.size(); }

private:
    std::vector<int> data_;
};

int main() {
    MyClass a(100), b(200);

    std::cout << "交换前: a=" << a.size() << ", b=" << b.size() << std::endl;

    // 使用 boost::swap（会调用自定义 swap）
    boost::swap(a, b);

    std::cout << "交换后: a=" << a.size() << ", b=" << b.size() << std::endl;

    return 0;
}
```

---

## base_from_member - 成员初始化辅助

```cpp
#include <boost/utility/base_from_member.hpp>
#include <iostream>
#include <fstream>

class Logger {
public:
    Logger(const std::string& filename) : file_(filename) {
        std::cout << "Logger 创建: " << filename << std::endl;
    }

    void log(const std::string& msg) {
        if (file_) {
            file_ << msg << std::endl;
        }
    }

private:
    std::ofstream file_;
};

// 使用 base_from_member 确保 logger_ 在其他成员之前初始化
class Application : private boost::base_from_member<Logger> {
    typedef boost::base_from_member<Logger> logger_base;

public:
    Application(const std::string& logfile)
        : logger_base(logfile)
        , name_("MyApp") {
        // 现在可以在构造函数体中使用 logger
        logger_base::member.log("应用程序启动");
    }

    void run() {
        logger_base::member.log("应用程序运行中");
    }

private:
    std::string name_;
};

int main() {
    Application app("app.log");
    app.run();

    return 0;
}
```

---

## BOOST_CURRENT_FUNCTION - 当前函数名

```cpp
#include <boost/current_function.hpp>
#include <iostream>

void foo() {
    std::cout << "当前函数: " << BOOST_CURRENT_FUNCTION << std::endl;
}

class MyClass {
public:
    void method() {
        std::cout << "当前函数: " << BOOST_CURRENT_FUNCTION << std::endl;
    }

    static void static_method() {
        std::cout << "当前函数: " << BOOST_CURRENT_FUNCTION << std::endl;
    }
};

int main() {
    std::cout << "当前函数: " << BOOST_CURRENT_FUNCTION << std::endl;

    foo();

    MyClass obj;
    obj.method();
    MyClass::static_method();

    return 0;
}
```

---

## result_of - 获取返回类型

```cpp
#include <boost/utility/result_of.hpp>
#include <iostream>
#include <string>

int add(int a, int b) {
    return a + b;
}

struct Multiplier {
    double operator()(double a, double b) const {
        return a * b;
    }
};

int main() {
    // 获取函数返回类型
    typedef boost::result_of<decltype(add)(int, int)>::type AddResult;
    AddResult result1 = add(3, 4);
    std::cout << "加法结果: " << result1 << std::endl;

    // 获取函数对象返回类型
    typedef boost::result_of<Multiplier(double, double)>::type MulResult;
    Multiplier mul;
    MulResult result2 = mul(3.5, 2.0);
    std::cout << "乘法结果: " << result2 << std::endl;

    return 0;
}
```

---

## string_ref - 字符串引用

```cpp
#include <boost/utility/string_ref.hpp>
#include <iostream>
#include <string>

void process_string(boost::string_ref str) {
    std::cout << "处理字符串: " << str << std::endl;
    std::cout << "长度: " << str.length() << std::endl;
    std::cout << "第一个字符: " << str.front() << std::endl;
}

int main() {
    // 从 std::string 创建
    std::string s = "Hello, World!";
    process_string(s);

    // 从 C 字符串创建
    process_string("Boost Utility");

    // 从字符数组创建
    char buffer[] = "Character Array";
    process_string(boost::string_ref(buffer, 9));  // 只取前9个字符

    return 0;
}
```

---

## value_init - 值初始化

```cpp
#include <boost/utility/value_init.hpp>
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    // 确保值初始化（即使是 POD 类型）
    boost::value_initialized<int> vi;
    std::cout << "值初始化的 int: " << vi.data() << std::endl;  // 0

    boost::value_initialized<Point> vp;
    std::cout << "Point: (" << vp.data().x << ", " << vp.data().y << ")" << std::endl;

    // 普通的 int 可能未初始化
    int normal_int;
    // std::cout << normal_int << std::endl;  // 未定义行为

    return 0;
}
```

---

## 实用宏

```cpp
#include <boost/version.hpp>
#include <boost/config.hpp>
#include <iostream>

int main() {
    // Boost 版本信息
    std::cout << "Boost 版本: " << BOOST_VERSION / 100000 << "."
              << BOOST_VERSION / 100 % 1000 << "."
              << BOOST_VERSION % 100 << std::endl;

    std::cout << "Boost 库版本: " << BOOST_LIB_VERSION << std::endl;

    // 编译器检测
#ifdef BOOST_MSVC
    std::cout << "编译器: MSVC" << std::endl;
#elif defined(BOOST_GCC)
    std::cout << "编译器: GCC" << std::endl;
#elif defined(BOOST_CLANG)
    std::cout << "编译器: Clang" << std::endl;
#endif

    // 平台检测
#ifdef BOOST_WINDOWS
    std::cout << "平台: Windows" << std::endl;
#elif defined(BOOST_LINUX)
    std::cout << "平台: Linux" << std::endl;
#elif defined(BOOST_MACOS)
    std::cout << "平台: macOS" << std::endl;
#endif

    return 0;
}
```

---

## 参考资源

- [Boost.Utility 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/utility/doc/html/index.html)
- [Boost.Core 文档](https://www.boost.org/doc/libs/1_90_0/libs/core/doc/html/index.html)

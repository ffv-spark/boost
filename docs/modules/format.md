# Boost.Format - 字符串格式化库

## 概述

Boost.Format 提供类型安全的 printf 风格字符串格式化。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <string>

int main() {
    // 基本格式化
    std::string result = (boost::format("Hello, %1%!") % "World").str();
    std::cout << result << std::endl;

    // 多个参数
    int age = 25;
    std::string name = "Alice";
    std::cout << boost::format("%1% is %2% years old") % name % age << std::endl;

    // 重复使用参数
    std::cout << boost::format("%1% + %1% = %2%") % 5 % 10 << std::endl;

    // 格式说明符
    std::cout << boost::format("Pi = %1$.2f") % 3.14159 << std::endl;
    std::cout << boost::format("Hex: 0x%1$04x") % 255 << std::endl;

    return 0;
}
```

---

## 格式说明符

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int num = 42;
    double pi = 3.14159265;

    // 整数格式
    std::cout << boost::format("Decimal: %1%") % num << std::endl;
    std::cout << boost::format("Hex: %1$x") % num << std::endl;
    std::cout << boost::format("Oct: %1$o") % num << std::endl;
    std::cout << boost::format("Width: %1$5d") % num << std::endl;
    std::cout << boost::format("Zero-pad: %1$05d") % num << std::endl;

    // 浮点格式
    std::cout << boost::format("Default: %1%") % pi << std::endl;
    std::cout << boost::format("Fixed: %1$.2f") % pi << std::endl;
    std::cout << boost::format("Scientific: %1$e") % pi << std::endl;
    std::cout << boost::format("Width: %1$10.3f") % pi << std::endl;

    // 字符串格式
    std::string str = "Hello";
    std::cout << boost::format("String: '%1%'") % str << std::endl;
    std::cout << boost::format("Width: '%1$10s'") % str << std::endl;
    std::cout << boost::format("Left: '%-1$10s'") % str << std::endl;

    return 0;
}
```

---

## 表格输出

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <vector>

struct Person {
    std::string name;
    int age;
    double salary;
};

int main() {
    std::vector<Person> people = {
        {"Alice", 25, 50000.0},
        {"Bob", 30, 60000.0},
        {"Charlie", 35, 75000.0}
    };

    // 表头
    boost::format header("%-15s %5s %12s");
    std::cout << header % "Name" % "Age" % "Salary" << std::endl;
    std::cout << std::string(35, '-') << std::endl;

    // 数据行
    boost::format row("%-15s %5d $%10.2f");
    for (const auto& p : people) {
        std::cout << row % p.name % p.age % p.salary << std::endl;
    }

    return 0;
}
```

---

## 日志格式化

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <string>
#include <ctime>

enum LogLevel { DEBUG, INFO, WARN, ERROR };

void log(LogLevel level, const std::string& message) {
    std::time_t now = std::time(nullptr);
    char timestamp[20];
    std::strftime(timestamp, sizeof(timestamp), "%Y-%m-%d %H:%M:%S",
                  std::localtime(&now));

    const char* level_str[] = {"DEBUG", "INFO", "WARN", "ERROR"};

    std::cout << boost::format("[%1%] [%2%] %3%")
                 % timestamp
                 % level_str[level]
                 % message
              << std::endl;
}

int main() {
    log(INFO, "Application started");
    log(DEBUG, boost::str(boost::format("Processing item %1%") % 42));
    log(WARN, "Memory usage high");
    log(ERROR, "Connection failed");

    return 0;
}
```

---

## 国际化支持

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int files = 5;

    // 中文
    std::cout << boost::format("找到 %1% 个文件") % files << std::endl;

    // 英文
    std::cout << boost::format("Found %1% files") % files << std::endl;

    // 日文
    std::cout << boost::format("%1% 個のファイルが見つかりました") % files << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **缓存格式对象**: 重复使用时缓存
2. **类型安全**: 比 printf 更安全
3. **性能**: 比流操作符慢，但更灵活
4. **C++20**: 考虑使用 std::format
5. **错误处理**: 捕获格式化异常

---

## 与其他方法对比

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <sstream>
#include <cstdio>

int main() {
    int x = 42;
    double y = 3.14;

    // printf（不安全）
    printf("x=%d, y=%.2f\n", x, y);

    // iostream（冗长）
    std::cout << "x=" << x << ", y=" << y << std::endl;

    // stringstream（冗长）
    std::stringstream ss;
    ss << "x=" << x << ", y=" << y;
    std::cout << ss.str() << std::endl;

    // boost::format（推荐）
    std::cout << boost::format("x=%1%, y=%2$.2f") % x % y << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Format 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/format/doc/format.html)

# Boost.Format - 格式化库

## 概述

Boost.Format 提供类型安全的字符串格式化功能，类似于 printf 但更安全。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int x = 42;
    double y = 3.14159;
    std::string name = "Alice";

    // 使用 boost::format
    std::string result = boost::str(
        boost::format("姓名: %1%, 整数: %2%, 浮点: %3%") % name % x % y
    );

    std::cout << result << std::endl;

    return 0;
}
```

---

## 基本格式化

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    // 位置参数
    std::cout << boost::format("%1% + %2% = %3%") % 2 % 3 % 5 << std::endl;

    // 重复使用参数
    std::cout << boost::format("%1% %1% %1%") % "Hello" << std::endl;

    // 改变参数顺序
    std::cout << boost::format("%2% %1%") % "World" % "Hello" << std::endl;

    return 0;
}
```

---

## 格式规范

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int num = 42;
    double pi = 3.14159265;

    // 整数格式
    std::cout << boost::format("十进制: %d") % num << std::endl;
    std::cout << boost::format("十六进制: %x") % num << std::endl;
    std::cout << boost::format("八进制: %o") % num << std::endl;

    // 浮点格式
    std::cout << boost::format("默认: %f") % pi << std::endl;
    std::cout << boost::format("科学计数: %e") % pi << std::endl;
    std::cout << boost::format("自动: %g") % pi << std::endl;

    return 0;
}
```

---

## 宽度和精度

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int num = 42;
    double pi = 3.14159265;

    // 宽度
    std::cout << boost::format("|%5d|") % num << std::endl;
    std::cout << boost::format("|%-5d|") % num << std::endl;  // 左对齐

    // 精度
    std::cout << boost::format("%.2f") % pi << std::endl;
    std::cout << boost::format("%.5f") % pi << std::endl;

    // 宽度和精度
    std::cout << boost::format("%10.3f") % pi << std::endl;

    return 0;
}
```

---

## 填充字符

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    int num = 42;

    // 零填充
    std::cout << boost::format("%05d") % num << std::endl;

    // 空格填充（默认）
    std::cout << boost::format("%5d") % num << std::endl;

    // 自定义填充
    boost::format fmt("%|1$=8|");  // 8字符宽，居中
    std::cout << fmt % num << std::endl;

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
        {"Alice", 30, 50000.00},
        {"Bob", 25, 45000.50},
        {"Charlie", 35, 60000.75}
    };

    // 表头
    boost::format header("%-10s %5s %10s");
    std::cout << header % "姓名" % "年龄" % "薪水" << std::endl;
    std::cout << std::string(28, '-') << std::endl;

    // 数据行
    boost::format row("%-10s %5d %10.2f");
    for (const auto& p : people) {
        std::cout << row % p.name % p.age % p.salary << std::endl;
    }

    return 0;
}
```

---

## 货币格式化

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<double> prices = {19.99, 299.95, 1499.00, 49.50};

    boost::format price_format("¥%|1$.2f|");

    std::cout << "商品价格:\n";
    for (size_t i = 0; i < prices.size(); ++i) {
        std::cout << "  商品 " << (i + 1) << ": " 
                  << price_format % prices[i] << std::endl;
    }

    return 0;
}
```

---

## 日期时间格式化

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <ctime>

int main() {
    std::time_t now = std::time(nullptr);
    std::tm* tm = std::localtime(&now);

    boost::format date_format("%04d-%02d-%02d %02d:%02d:%02d");

    std::string datetime = boost::str(
        date_format 
            % (tm->tm_year + 1900)
            % (tm->tm_mon + 1)
            % tm->tm_mday
            % tm->tm_hour
            % tm->tm_min
            % tm->tm_sec
    );

    std::cout << "当前时间: " << datetime << std::endl;

    return 0;
}
```

---

## 日志格式化

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <string>

enum LogLevel { DEBUG, INFO, WARNING, ERROR };

void log(LogLevel level, const std::string& message) {
    const char* level_str[] = {"DEBUG", "INFO", "WARN", "ERROR"};

    boost::format log_format("[%1%] %|2$-8| %3%");

    // 简单的时间戳
    static int counter = 0;

    std::cout << log_format % counter++ % level_str[level] % message << std::endl;
}

int main() {
    log(INFO, "应用程序启动");
    log(DEBUG, "加载配置文件");
    log(INFO, "连接到数据库");
    log(WARNING, "内存使用率较高");
    log(ERROR, "无法连接到服务器");

    return 0;
}
```

---

## 错误处理

```cpp
#include <boost/format.hpp>
#include <iostream>

int main() {
    try {
        // 参数不足
        boost::format fmt("%1% %2% %3%");
        fmt % "first" % "second";  // 缺少第三个参数

        std::cout << fmt << std::endl;

    } catch (const boost::io::too_few_args& e) {
        std::cout << "参数不足: " << e.what() << std::endl;
    }

    try {
        // 参数过多
        boost::format fmt("%1%");
        fmt % "first" % "second";  // 多余的参数

        std::cout << fmt << std::endl;

    } catch (const boost::io::too_many_args& e) {
        std::cout << "参数过多: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 本地化

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <locale>

int main() {
    // 中文千分位分隔符
    std::locale::global(std::locale(""));

    double large_number = 1234567.89;

    boost::format fmt("%|1$,.2f|");
    fmt.imbue(std::locale());

    std::cout << "格式化数字: " << fmt % large_number << std::endl;

    return 0;
}
```

---

## 与 printf 对比

```cpp
#include <boost/format.hpp>
#include <iostream>
#include <cstdio>

int main() {
    int x = 42;
    double y = 3.14;
    std::string name = "Alice";

    // printf（不安全）
    std::printf("printf: %s, %d, %.2f\n", name.c_str(), x, y);

    // boost::format（类型安全）
    std::cout << boost::format("format: %1%, %2%, %.2f") % name % x % y << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Format 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/format/doc/format.html)

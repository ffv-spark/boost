# Boost.LexicalCast - 词法转换库

## 概述

Boost.LexicalCast 提供简单高效的类型转换，特别是字符串与数值之间的转换。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;
    using boost::bad_lexical_cast;

    // 字符串转整数
    int i = lexical_cast<int>("123");
    std::cout << "整数: " << i << std::endl;

    // 整数转字符串
    std::string s = lexical_cast<std::string>(456);
    std::cout << "字符串: " << s << std::endl;

    // 浮点数转换
    double d = lexical_cast<double>("3.14159");
    std::cout << "浮点数: " << d << std::endl;

    return 0;
}
```

---

## 基本类型转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;

    // 数值转字符串
    std::string s1 = lexical_cast<std::string>(42);
    std::string s2 = lexical_cast<std::string>(3.14);
    std::string s3 = lexical_cast<std::string>(true);

    std::cout << "42 -> \"" << s1 << "\"" << std::endl;
    std::cout << "3.14 -> \"" << s2 << "\"" << std::endl;
    std::cout << "true -> \"" << s3 << "\"" << std::endl;

    // 字符串转数值
    int i = lexical_cast<int>("100");
    double d = lexical_cast<double>("2.718");
    bool b = lexical_cast<bool>("1");

    std::cout << "\"100\" -> " << i << std::endl;
    std::cout << "\"2.718\" -> " << d << std::endl;
    std::cout << "\"1\" -> " << std::boolalpha << b << std::endl;

    return 0;
}
```

---

## 异常处理

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;
    using boost::bad_lexical_cast;

    try {
        // 有效转换
        int i = lexical_cast<int>("123");
        std::cout << "成功: " << i << std::endl;

        // 无效转换
        int j = lexical_cast<int>("abc");
    } catch (const bad_lexical_cast& e) {
        std::cout << "转换失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 数值范围检查

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;
    using boost::bad_lexical_cast;

    try {
        // 转换超出范围的数字
        short s = lexical_cast<short>("100000");
    } catch (const bad_lexical_cast& e) {
        std::cout << "数值超出范围: " << e.what() << std::endl;
    }

    // 正常范围
    try {
        short s = lexical_cast<short>("32000");
        std::cout << "在范围内: " << s << std::endl;
    } catch (const bad_lexical_cast& e) {
        std::cout << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 解析用户输入

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;
    using boost::bad_lexical_cast;

    std::cout << "请输入一个整数: ";
    std::string input;
    std::getline(std::cin, input);

    try {
        int number = lexical_cast<int>(input);
        std::cout << "您输入了: " << number << std::endl;
        std::cout << "平方: " << number * number << std::endl;
    } catch (const bad_lexical_cast&) {
        std::cout << "无效的整数" << std::endl;
    }

    return 0;
}
```

---

## 与容器配合

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    using boost::lexical_cast;

    std::vector<std::string> string_numbers = {"1", "2", "3", "4", "5"};
    std::vector<int> numbers;

    // 批量转换
    for (const auto& s : string_numbers) {
        numbers.push_back(lexical_cast<int>(s));
    }

    std::cout << "转换后的数字: ";
    for (int n : numbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 自定义类型支持

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <sstream>

struct Point {
    int x, y;
};

// 输出流操作符（lexical_cast 需要）
std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << p.x << "," << p.y;
    return os;
}

// 输入流操作符（lexical_cast 需要）
std::istream& operator>>(std::istream& is, Point& p) {
    char comma;
    is >> p.x >> comma >> p.y;
    return is;
}

int main() {
    using boost::lexical_cast;

    // Point 转字符串
    Point p1{10, 20};
    std::string s = lexical_cast<std::string>(p1);
    std::cout << "Point 转字符串: " << s << std::endl;

    // 字符串转 Point
    Point p2 = lexical_cast<Point>("30,40");
    std::cout << "字符串转 Point: (" << p2.x << ", " << p2.y << ")" << std::endl;

    return 0;
}
```

---

## 性能对比

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <sstream>
#include <chrono>

int main() {
    const int iterations = 100000;

    // 使用 lexical_cast
    auto start1 = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        std::string s = boost::lexical_cast<std::string>(i);
    }
    auto end1 = std::chrono::high_resolution_clock::now();

    // 使用 stringstream
    auto start2 = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        std::stringstream ss;
        ss << i;
        std::string s = ss.str();
    }
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration_cast<std::chrono::milliseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::milliseconds>(end2 - start2);

    std::cout << "lexical_cast: " << duration1.count() << " ms" << std::endl;
    std::cout << "stringstream: " << duration2.count() << " ms" << std::endl;

    return 0;
}
```

---

## 格式化控制

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <iomanip>

int main() {
    using boost::lexical_cast;

    double d = 3.14159265358979;

    // lexical_cast 使用默认精度
    std::string s1 = lexical_cast<std::string>(d);
    std::cout << "默认: " << s1 << std::endl;

    // 需要更多控制时使用 stringstream
    std::stringstream ss;
    ss << std::fixed << std::setprecision(2) << d;
    std::string s2 = ss.str();
    std::cout << "控制精度: " << s2 << std::endl;

    return 0;
}
```

---

## 布尔值转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::lexical_cast;
    using boost::bad_lexical_cast;

    // 布尔值转字符串
    std::string s1 = lexical_cast<std::string>(true);
    std::string s2 = lexical_cast<std::string>(false);
    std::cout << "true -> \"" << s1 << "\"" << std::endl;
    std::cout << "false -> \"" << s2 << "\"" << std::endl;

    // 字符串转布尔值
    try {
        bool b1 = lexical_cast<bool>("1");
        bool b2 = lexical_cast<bool>("0");
        bool b3 = lexical_cast<bool>("true");
        bool b4 = lexical_cast<bool>("false");

        std::cout << std::boolalpha;
        std::cout << "\"1\" -> " << b1 << std::endl;
        std::cout << "\"0\" -> " << b2 << std::endl;
        std::cout << "\"true\" -> " << b3 << std::endl;
        std::cout << "\"false\" -> " << b4 << std::endl;
    } catch (const bad_lexical_cast& e) {
        std::cout << "转换失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## CSV 解析示例

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <vector>
#include <sstream>

struct Record {
    std::string name;
    int age;
    double salary;
};

Record parse_csv_line(const std::string& line) {
    using boost::lexical_cast;

    std::vector<std::string> fields;
    std::stringstream ss(line);
    std::string field;

    while (std::getline(ss, field, ',')) {
        fields.push_back(field);
    }

    Record record;
    record.name = fields[0];
    record.age = lexical_cast<int>(fields[1]);
    record.salary = lexical_cast<double>(fields[2]);

    return record;
}

int main() {
    std::vector<std::string> csv_lines = {
        "Alice,30,50000.00",
        "Bob,25,45000.50",
        "Charlie,35,60000.75"
    };

    std::cout << "CSV 解析结果:\n";
    for (const auto& line : csv_lines) {
        try {
            Record rec = parse_csv_line(line);
            std::cout << "  " << rec.name << ", "
                      << rec.age << "岁, "
                      << rec.salary << "元" << std::endl;
        } catch (const boost::bad_lexical_cast& e) {
            std::cout << "  解析失败: " << line << std::endl;
        }
    }

    return 0;
}
```

---

## 参考资源

- [Boost.LexicalCast 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_lexical_cast.html)
- [C++ 类型转换](https://en.cppreference.com/w/cpp/string/basic_string/to_string)

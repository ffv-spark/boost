# Boost.Conversion - 类型转换库

## 概述

Boost.Conversion 提供安全的类型转换工具，包括 lexical_cast、numeric_cast 等。

**类型**: 仅头文件库

---

## lexical_cast - 字符串转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    // 字符串转数字
    std::string str_num = "12345";
    int num = boost::lexical_cast<int>(str_num);
    std::cout << "字符串 \"" << str_num << "\" 转为整数: " << num << std::endl;

    // 数字转字符串
    double pi = 3.14159;
    std::string str_pi = boost::lexical_cast<std::string>(pi);
    std::cout << "浮点数 " << pi << " 转为字符串: \"" << str_pi << "\"" << std::endl;

    // 处理转换失败
    try {
        int invalid = boost::lexical_cast<int>("not a number");
    } catch (const boost::bad_lexical_cast& e) {
        std::cout << "转换失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## numeric_cast - 数值转换

```cpp
#include <boost/numeric/conversion/cast.hpp>
#include <iostream>
#include <limits>

int main() {
    // 安全的数值转换
    int large_int = 10000;
    short small_short = boost::numeric_cast<short>(large_int);
    std::cout << "int " << large_int << " 转为 short: " << small_short << std::endl;

    // 检测溢出
    try {
        int too_large = std::numeric_limits<short>::max() + 1;
        short result = boost::numeric_cast<short>(too_large);
    } catch (const boost::numeric::positive_overflow& e) {
        std::cout << "正溢出: " << e.what() << std::endl;
    }

    // 检测下溢
    try {
        int too_small = std::numeric_limits<short>::min() - 1;
        short result = boost::numeric_cast<short>(too_small);
    } catch (const boost::numeric::negative_overflow& e) {
        std::cout << "负溢出: " << e.what() << std::endl;
    }

    // 浮点到整数转换
    try {
        double d = 3.99;
        int i = boost::numeric_cast<int>(d);
        std::cout << "double " << d << " 转为 int: " << i << std::endl;
    } catch (const boost::numeric::bad_numeric_cast& e) {
        std::cout << "转换错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 批量转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> str_numbers = {"10", "20", "30", "40", "50"};
    std::vector<int> numbers;

    // 转换字符串列表为整数列表
    for (const auto& str : str_numbers) {
        try {
            numbers.push_back(boost::lexical_cast<int>(str));
        } catch (const boost::bad_lexical_cast&) {
            std::cout << "跳过无效值: " << str << std::endl;
        }
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

## 自定义类型转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <sstream>
#include <string>

class Point {
public:
    int x, y;

    Point() : x(0), y(0) {}
    Point(int x_, int y_) : x(x_), y(y_) {}

    // 重载输出流运算符
    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        os << "(" << p.x << "," << p.y << ")";
        return os;
    }

    // 重载输入流运算符
    friend std::istream& operator>>(std::istream& is, Point& p) {
        char lparen, comma, rparen;
        is >> lparen >> p.x >> comma >> p.y >> rparen;
        return is;
    }
};

int main() {
    Point p1(10, 20);

    // Point 转字符串
    std::string str = boost::lexical_cast<std::string>(p1);
    std::cout << "Point 转字符串: " << str << std::endl;

    // 字符串转 Point
    try {
        Point p2 = boost::lexical_cast<Point>("(30,40)");
        std::cout << "字符串转 Point: (" << p2.x << "," << p2.y << ")" << std::endl;
    } catch (const boost::bad_lexical_cast& e) {
        std::cout << "转换失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 性能优化转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <chrono>
#include <sstream>

int main() {
    const int iterations = 100000;

    // 使用 lexical_cast
    auto start1 = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        std::string s = boost::lexical_cast<std::string>(i);
    }
    auto end1 = std::chrono::high_resolution_clock::now();
    auto duration1 = std::chrono::duration_cast<std::chrono::milliseconds>(end1 - start1);

    // 使用 stringstream
    auto start2 = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        std::stringstream ss;
        ss << i;
        std::string s = ss.str();
    }
    auto end2 = std::chrono::high_resolution_clock::now();
    auto duration2 = std::chrono::duration_cast<std::chrono::milliseconds>(end2 - start2);

    std::cout << "lexical_cast 耗时: " << duration1.count() << " ms" << std::endl;
    std::cout << "stringstream 耗时: " << duration2.count() << " ms" << std::endl;

    return 0;
}
```

---

## 范围检查转换

```cpp
#include <boost/numeric/conversion/cast.hpp>
#include <boost/numeric/conversion/bounds.hpp>
#include <iostream>
#include <limits>

template<typename Target, typename Source>
bool safe_convert(Source value, Target& result) {
    try {
        result = boost::numeric_cast<Target>(value);
        return true;
    } catch (const boost::numeric::bad_numeric_cast&) {
        return false;
    }
}

int main() {
    int large_value = 100000;
    short result;

    if (safe_convert(large_value, result)) {
        std::cout << "转换成功: " << result << std::endl;
    } else {
        std::cout << "转换失败：值超出范围" << std::endl;
    }

    // 检查边界
    std::cout << "short 最小值: " << boost::numeric::bounds<short>::lowest() << std::endl;
    std::cout << "short 最大值: " << boost::numeric::bounds<short>::highest() << std::endl;

    return 0;
}
```

---

## 转换特性检查

```cpp
#include <boost/numeric/conversion/converter_policies.hpp>
#include <boost/numeric/conversion/cast.hpp>
#include <iostream>

int main() {
    // 使用自定义溢出处理策略
    typedef boost::numeric::converter<
        short,
        int,
        boost::numeric::conversion_traits<short, int>,
        boost::numeric::def_overflow_handler,  // 默认抛出异常
        boost::numeric::Trunc<int>             // 截断策略
    > Int2Short;

    int value = 1000;

    try {
        short result = Int2Short::convert(value);
        std::cout << "转换结果: " << result << std::endl;
    } catch (const boost::numeric::positive_overflow&) {
        std::cout << "值太大，无法转换" << std::endl;
    }

    return 0;
}
```

---

## 浮点转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <iomanip>

int main() {
    // 浮点数转字符串
    double pi = 3.141592653589793;
    std::string str_pi = boost::lexical_cast<std::string>(pi);
    std::cout << "默认精度: " << str_pi << std::endl;

    // 字符串转浮点数
    std::string scientific = "1.23e-4";
    double value = boost::lexical_cast<double>(scientific);
    std::cout << "科学计数法: " << std::setprecision(10) << value << std::endl;

    // 处理特殊值
    try {
        double inf = boost::lexical_cast<double>("inf");
        std::cout << "无穷大: " << inf << std::endl;
    } catch (const boost::bad_lexical_cast& e) {
        std::cout << "转换失败: " << e.what() << std::endl;
    }

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
    // 布尔值转字符串
    bool flag = true;
    std::string str_flag = boost::lexical_cast<std::string>(flag);
    std::cout << "true 转字符串: \"" << str_flag << "\"" << std::endl;

    // 字符串转布尔值
    try {
        bool b1 = boost::lexical_cast<bool>("1");
        bool b2 = boost::lexical_cast<bool>("0");
        std::cout << "\"1\" 转布尔: " << b1 << std::endl;
        std::cout << "\"0\" 转布尔: " << b2 << std::endl;

        // 注意：非 "0" 和 "1" 的字符串会抛出异常
        bool b3 = boost::lexical_cast<bool>("true");
    } catch (const boost::bad_lexical_cast& e) {
        std::cout << "布尔转换失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 容错转换

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <vector>

template<typename T>
T safe_lexical_cast(const std::string& str, const T& default_value) {
    try {
        return boost::lexical_cast<T>(str);
    } catch (const boost::bad_lexical_cast&) {
        return default_value;
    }
}

int main() {
    std::vector<std::string> inputs = {"123", "abc", "456", "xyz", "789"};

    std::cout << "转换结果（无效值用 -1 代替）:\n";
    for (const auto& input : inputs) {
        int value = safe_lexical_cast(input, -1);
        std::cout << "\"" << input << "\" -> " << value << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.LexicalCast 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_lexical_cast.html)
- [Boost.NumericConversion 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/numeric/conversion/doc/html/index.html)

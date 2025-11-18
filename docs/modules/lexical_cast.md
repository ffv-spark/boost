# Boost.Lexical_Cast - 类型转换库

## 概述

Boost.Lexical_Cast 提供了简单安全的字符串与其他类型之间的转换。

**类型**: 仅头文件库

**主要特性**:
- 简洁的转换语法
- 类型安全
- 异常处理
- 自动类型推导

---

## 快速开始

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    // 字符串转整数
    int num = boost::lexical_cast<int>("12345");
    std::cout << "整数: " << num << std::endl;

    // 整数转字符串
    std::string str = boost::lexical_cast<std::string>(42);
    std::cout << "字符串: " << str << std::endl;

    // 字符串转浮点数
    double pi = boost::lexical_cast<double>("3.14159");
    std::cout << "浮点数: " << pi << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -o example
```

---

## 基本用法

### 数字与字符串互转

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    // 1. 字符串 -> 整数
    int i1 = boost::lexical_cast<int>("123");
    int i2 = boost::lexical_cast<int>("-456");

    // 2. 字符串 -> 浮点数
    float f = boost::lexical_cast<float>("3.14");
    double d = boost::lexical_cast<double>("2.71828");

    // 3. 整数 -> 字符串
    std::string s1 = boost::lexical_cast<std::string>(100);
    std::string s2 = boost::lexical_cast<std::string>(-200);

    // 4. 浮点数 -> 字符串
    std::string s3 = boost::lexical_cast<std::string>(3.14159);

    std::cout << "整数: " << i1 << ", " << i2 << std::endl;
    std::cout << "浮点: " << f << ", " << d << std::endl;
    std::cout << "字符串: " << s1 << ", " << s2 << ", " << s3 << std::endl;

    return 0;
}
```

### 错误处理

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    try {
        // 有效转换
        int valid = boost::lexical_cast<int>("123");
        std::cout << "有效: " << valid << std::endl;

        // 无效转换 - 抛出异常
        int invalid = boost::lexical_cast<int>("abc");

    } catch (const boost::bad_lexical_cast& e) {
        std::cerr << "转换失败: " << e.what() << std::endl;
    }

    // 使用 try_lexical_convert （不抛异常）
    int result;
    if (boost::conversion::try_lexical_convert("456", result)) {
        std::cout << "转换成功: " << result << std::endl;
    } else {
        std::cout << "转换失败" << std::endl;
    }

    if (boost::conversion::try_lexical_convert("xyz", result)) {
        std::cout << "转换成功: " << result << std::endl;
    } else {
        std::cout << "转换失败" << std::endl;
    }

    return 0;
}
```

---

## 常见类型转换

### 布尔值

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>

int main() {
    // 整数 -> 布尔
    bool b1 = boost::lexical_cast<bool>("1");   // true
    bool b2 = boost::lexical_cast<bool>("0");   // false

    // 布尔 -> 字符串
    std::string s1 = boost::lexical_cast<std::string>(true);   // "1"
    std::string s2 = boost::lexical_cast<std::string>(false);  // "0"

    std::cout << std::boolalpha;
    std::cout << "b1: " << b1 << ", b2: " << b2 << std::endl;
    std::cout << "s1: " << s1 << ", s2: " << s2 << std::endl;

    return 0;
}
```

### 字符和字符串

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>

int main() {
    // 字符 -> 整数
    int ascii_a = boost::lexical_cast<int>('A');  // 65

    // 单字符字符串 -> 字符
    char ch = boost::lexical_cast<char>("X");

    // 字符 -> 字符串
    std::string s = boost::lexical_cast<std::string>('Z');

    std::cout << "ASCII 'A': " << ascii_a << std::endl;
    std::cout << "字符: " << ch << std::endl;
    std::cout << "字符串: " << s << std::endl;

    return 0;
}
```

---

## 实用示例

### 命令行参数解析

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <map>

class ArgumentParser {
public:
    ArgumentParser(int argc, char* argv[]) {
        for (int i = 1; i < argc; ++i) {
            std::string arg = argv[i];
            size_t pos = arg.find('=');

            if (pos != std::string::npos) {
                std::string key = arg.substr(0, pos);
                std::string value = arg.substr(pos + 1);
                args_[key] = value;
            }
        }
    }

    template<typename T>
    T get(const std::string& key, const T& default_value) const {
        auto it = args_.find(key);
        if (it != args_.end()) {
            try {
                return boost::lexical_cast<T>(it->second);
            } catch (const boost::bad_lexical_cast&) {
                return default_value;
            }
        }
        return default_value;
    }

private:
    std::map<std::string, std::string> args_;
};

int main(int argc, char* argv[]) {
    ArgumentParser parser(argc, argv);

    int port = parser.get<int>("port", 8080);
    std::string host = parser.get<std::string>("host", "localhost");
    bool debug = parser.get<bool>("debug", false);

    std::cout << "主机: " << host << std::endl;
    std::cout << "端口: " << port << std::endl;
    std::cout << "调试: " << (debug ? "开启" : "关闭") << std::endl;

    return 0;
}

// 使用: ./program host=127.0.0.1 port=9000 debug=1
```

### CSV 解析

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <sstream>
#include <vector>
#include <string>

struct Record {
    int id;
    std::string name;
    double score;
};

std::vector<std::string> split(const std::string& str, char delim) {
    std::vector<std::string> result;
    std::stringstream ss(str);
    std::string item;

    while (std::getline(ss, item, delim)) {
        result.push_back(item);
    }

    return result;
}

Record parse_csv_line(const std::string& line) {
    auto fields = split(line, ',');

    Record record;

    try {
        record.id = boost::lexical_cast<int>(fields[0]);
        record.name = fields[1];
        record.score = boost::lexical_cast<double>(fields[2]);
    } catch (const boost::bad_lexical_cast& e) {
        std::cerr << "解析错误: " << e.what() << std::endl;
        throw;
    }

    return record;
}

int main() {
    std::vector<std::string> csv_lines = {
        "1,Alice,95.5",
        "2,Bob,87.3",
        "3,Charlie,92.1"
    };

    std::vector<Record> records;

    for (const auto& line : csv_lines) {
        try {
            records.push_back(parse_csv_line(line));
        } catch (...) {
            std::cerr << "跳过无效行: " << line << std::endl;
        }
    }

    std::cout << "解析的记录:\n";
    for (const auto& r : records) {
        std::cout << "ID: " << r.id
                  << ", 姓名: " << r.name
                  << ", 分数: " << r.score << std::endl;
    }

    return 0;
}
```

### 配置文件读取

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <fstream>
#include <map>
#include <string>

class ConfigReader {
public:
    bool load(const std::string& filename) {
        std::ifstream file(filename);
        if (!file) return false;

        std::string line;
        while (std::getline(file, line)) {
            // 跳过注释和空行
            if (line.empty() || line[0] == '#') continue;

            size_t pos = line.find('=');
            if (pos != std::string::npos) {
                std::string key = line.substr(0, pos);
                std::string value = line.substr(pos + 1);

                // 去除空格
                key.erase(0, key.find_first_not_of(" \t"));
                key.erase(key.find_last_not_of(" \t") + 1);
                value.erase(0, value.find_first_not_of(" \t"));
                value.erase(value.find_last_not_of(" \t") + 1);

                config_[key] = value;
            }
        }

        return true;
    }

    template<typename T>
    T get(const std::string& key, const T& default_value) const {
        auto it = config_.find(key);
        if (it != config_.end()) {
            try {
                return boost::lexical_cast<T>(it->second);
            } catch (const boost::bad_lexical_cast&) {
                return default_value;
            }
        }
        return default_value;
    }

private:
    std::map<std::string, std::string> config_;
};

int main() {
    // config.txt 内容:
    // server.host = localhost
    // server.port = 8080
    // database.pool_size = 10
    // logging.enabled = 1

    ConfigReader config;

    if (config.load("config.txt")) {
        std::string host = config.get<std::string>("server.host", "0.0.0.0");
        int port = config.get<int>("server.port", 80);
        int pool_size = config.get<int>("database.pool_size", 5);
        bool logging = config.get<bool>("logging.enabled", false);

        std::cout << "服务器: " << host << ":" << port << std::endl;
        std::cout << "连接池大小: " << pool_size << std::endl;
        std::cout << "日志: " << (logging ? "启用" : "禁用") << std::endl;
    } else {
        std::cerr << "无法加载配置文件" << std::endl;
    }

    return 0;
}
```

### 数据验证

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <regex>

class Validator {
public:
    static bool is_valid_int(const std::string& str) {
        try {
            boost::lexical_cast<int>(str);
            return true;
        } catch (const boost::bad_lexical_cast&) {
            return false;
        }
    }

    static bool is_valid_double(const std::string& str) {
        try {
            boost::lexical_cast<double>(str);
            return true;
        } catch (const boost::bad_lexical_cast&) {
            return false;
        }
    }

    static bool is_in_range(const std::string& str, int min, int max) {
        try {
            int value = boost::lexical_cast<int>(str);
            return value >= min && value <= max;
        } catch (const boost::bad_lexical_cast&) {
            return false;
        }
    }
};

int main() {
    std::vector<std::string> test_values = {
        "123", "abc", "3.14", "-456", "999999999999999999"
    };

    for (const auto& val : test_values) {
        std::cout << "\"" << val << "\": ";

        if (Validator::is_valid_int(val)) {
            std::cout << "有效整数";
            if (Validator::is_in_range(val, 0, 1000)) {
                std::cout << " (在范围内)";
            }
        } else if (Validator::is_valid_double(val)) {
            std::cout << "有效浮点数";
        } else {
            std::cout << "无效数字";
        }

        std::cout << std::endl;
    }

    return 0;
}
```

---

## 性能考虑

```cpp
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <sstream>
#include <chrono>

void benchmark_lexical_cast() {
    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < 100000; ++i) {
        std::string s = boost::lexical_cast<std::string>(i);
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "lexical_cast: " << duration.count() << " ms" << std::endl;
}

void benchmark_stringstream() {
    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < 100000; ++i) {
        std::stringstream ss;
        ss << i;
        std::string s = ss.str();
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "stringstream: " << duration.count() << " ms" << std::endl;
}

void benchmark_to_string() {
    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < 100000; ++i) {
        std::string s = std::to_string(i);
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "std::to_string: " << duration.count() << " ms" << std::endl;
}

int main() {
    benchmark_lexical_cast();
    benchmark_stringstream();
    benchmark_to_string();

    return 0;
}
```

---

## 最佳实践

1. **错误处理**: 始终使用 try-catch 或 try_lexical_convert
2. **性能**: 频繁转换时考虑使用 std::to_string
3. **类型安全**: 优于 atoi 等 C 风格函数
4. **可读性**: 代码更简洁明了
5. **C++11+**: 结合 std::to_string 使用

---

## 与其他方法的对比

| 方法 | 优点 | 缺点 |
|------|------|------|
| `lexical_cast` | 简洁、类型安全 | 相对较慢 |
| `std::stringstream` | 灵活、格式化 | 冗长、慢 |
| `std::to_string` (C++11) | 快速 | 仅支持数字转字符串 |
| `std::stoi` (C++11) | 快速 | 仅支持字符串转整数 |
| `atoi` | 快速 | 不安全、无错误处理 |

---

## 参考资源

- [Boost.Lexical_Cast 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_lexical_cast.html)

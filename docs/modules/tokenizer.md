# Boost.Tokenizer - 标记化库

## 概述

Boost.Tokenizer 提供将字符串分割成标记（tokens）的功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Hello,World,Boost,C++";

    // 使用逗号分隔
    boost::tokenizer<> tok(text);

    std::cout << "标记:\n";
    for (const auto& token : tok) {
        std::cout << "  " << token << std::endl;
    }

    return 0;
}
```

---

## 自定义分隔符

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "apple;banana|orange:grape";

    // 使用多个分隔符
    typedef boost::tokenizer<boost::char_separator<char>> tokenizer;
    boost::char_separator<char> sep(";|:");

    tokenizer tok(text, sep);

    std::cout << "水果列表:\n";
    for (const auto& token : tok) {
        std::cout << "  " << token << std::endl;
    }

    return 0;
}
```

---

## 空格分隔

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "  The  quick   brown  fox  ";

    // 忽略空标记
    boost::char_separator<char> sep(" ");
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    std::cout << "单词:\n";
    for (const auto& token : tok) {
        std::cout << "  [" << token << "]" << std::endl;
    }

    return 0;
}
```

---

## 保留分隔符

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "x+y-z*2";

    // 保留分隔符
    boost::char_separator<char> sep("", "+-*");
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    std::cout << "表达式标记:\n";
    for (const auto& token : tok) {
        std::cout << "  [" << token << "]" << std::endl;
    }

    return 0;
}
```

---

## CSV 解析

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::string csv_line = "Alice,30,Engineer,\"New York, NY\"";

    // escaped_list_separator 处理引号和转义
    typedef boost::tokenizer<boost::escaped_list_separator<char>> tokenizer;
    tokenizer tok(csv_line);

    std::vector<std::string> fields;
    for (const auto& token : tok) {
        fields.push_back(token);
    }

    std::cout << "CSV 字段:\n";
    for (size_t i = 0; i < fields.size(); ++i) {
        std::cout << "  字段 " << i << ": [" << fields[i] << "]" << std::endl;
    }

    return 0;
}
```

---

## 命令行解析

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string command = "gcc -o output file1.c file2.c -lm";

    boost::char_separator<char> sep(" ");
    boost::tokenizer<boost::char_separator<char>> tok(command, sep);

    std::cout << "命令行参数:\n";
    int i = 0;
    for (const auto& token : tok) {
        std::cout << "  argv[" << i++ << "]: " << token << std::endl;
    }

    return 0;
}
```

---

## 单词计数

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>
#include <map>
#include <algorithm>

int main() {
    std::string text = "the quick brown fox jumps over the lazy dog the fox";

    boost::char_separator<char> sep(" ");
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    std::map<std::string, int> word_count;

    for (auto token : tok) {
        // 转换为小写
        std::transform(token.begin(), token.end(), token.begin(), ::tolower);
        ++word_count[token];
    }

    std::cout << "单词频率:\n";
    for (const auto& p : word_count) {
        std::cout << "  " << p.first << ": " << p.second << std::endl;
    }

    return 0;
}
```

---

## 路径分割

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string path = "/usr/local/bin/program";

    boost::char_separator<char> sep("/");
    boost::tokenizer<boost::char_separator<char>> tok(path, sep);

    std::cout << "路径组件:\n";
    for (const auto& token : tok) {
        if (!token.empty()) {
            std::cout << "  " << token << std::endl;
        }
    }

    return 0;
}
```

---

## 邮件地址解析

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

struct Email {
    std::string username;
    std::string domain;
};

Email parse_email(const std::string& email) {
    boost::char_separator<char> sep("@");
    boost::tokenizer<boost::char_separator<char>> tok(email, sep);

    auto it = tok.begin();
    Email result;
    result.username = *it++;
    if (it != tok.end()) {
        result.domain = *it;
    }

    return result;
}

int main() {
    std::string email = "user@example.com";
    Email parsed = parse_email(email);

    std::cout << "邮箱: " << email << std::endl;
    std::cout << "用户名: " << parsed.username << std::endl;
    std::cout << "域名: " << parsed.domain << std::endl;

    return 0;
}
```

---

## 多行文本处理

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Line 1\nLine 2\nLine 3\nLine 4";

    boost::char_separator<char> sep("\n");
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    std::cout << "行:\n";
    int line_num = 1;
    for (const auto& line : tok) {
        std::cout << line_num++ << ": " << line << std::endl;
    }

    return 0;
}
```

---

## 配置文件解析

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>
#include <map>

int main() {
    std::string config = "host=localhost\nport=8080\ntimeout=30";

    std::map<std::string, std::string> settings;

    // 按行分割
    boost::char_separator<char> line_sep("\n");
    boost::tokenizer<boost::char_separator<char>> lines(config, line_sep);

    for (const auto& line : lines) {
        // 按等号分割
        boost::char_separator<char> eq_sep("=");
        boost::tokenizer<boost::char_separator<char>> parts(line, eq_sep);

        auto it = parts.begin();
        std::string key = *it++;
        if (it != parts.end()) {
            std::string value = *it;
            settings[key] = value;
        }
    }

    std::cout << "配置:\n";
    for (const auto& p : settings) {
        std::cout << "  " << p.first << " = " << p.second << std::endl;
    }

    return 0;
}
```

---

## offset_separator

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::string data = "20231125Alice   Engineer";

    // 固定宽度字段：8字符日期，8字符姓名，剩余为职位
    std::vector<int> offsets = {8, 8};

    boost::offset_separator sep(offsets.begin(), offsets.end());
    boost::tokenizer<boost::offset_separator> tok(data, sep);

    std::cout << "固定宽度字段:\n";
    int field = 1;
    for (const auto& token : tok) {
        std::cout << "  字段 " << field++ << ": [" << token << "]" << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Tokenizer 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/tokenizer/index.html)

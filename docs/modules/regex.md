# Boost.Regex - 正则表达式库

## 概述

Boost.Regex 提供了强大的正则表达式匹配、搜索和替换功能。

**类型**: 需要编译链接的库

**链接库**: `-lboost_regex`

**注意**: C++11 已纳入标准库 (`std::regex`)，但 Boost.Regex 功能更丰富

---

## 快速开始

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Email: test@example.com";
    boost::regex pattern(R"((\w+)@(\w+\.\w+))");

    if (boost::regex_search(text, pattern)) {
        std::cout << "找到匹配的邮箱" << std::endl;
    }

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_regex -o example
```

---

## 基本匹配

### regex_match - 完整匹配

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    // 完整匹配电话号码
    boost::regex phone_pattern(R"(\d{3}-\d{4}-\d{4})");

    std::string phone1 = "138-1234-5678";
    std::string phone2 = "电话：138-1234-5678";

    std::cout << "phone1 匹配: "
              << boost::regex_match(phone1, phone_pattern) << std::endl; // true
    std::cout << "phone2 匹配: "
              << boost::regex_match(phone2, phone_pattern) << std::endl; // false

    return 0;
}
```

### regex_search - 部分匹配

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "联系方式：Email: user@example.com, 电话：138-1234-5678";

    // 搜索邮箱
    boost::regex email_pattern(R"(\w+@\w+\.\w+)");
    boost::smatch match;

    if (boost::regex_search(text, match, email_pattern)) {
        std::cout << "找到邮箱: " << match[0] << std::endl;
        std::cout << "位置: " << match.position() << std::endl;
        std::cout << "长度: " << match.length() << std::endl;
    }

    return 0;
}
```

---

## 捕获组

### 提取子匹配

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "2024-03-15";

    // 使用捕获组提取年月日
    boost::regex date_pattern(R"((\d{4})-(\d{2})-(\d{2}))");
    boost::smatch match;

    if (boost::regex_match(text, match, date_pattern)) {
        std::cout << "完整日期: " << match[0] << std::endl;
        std::cout << "年: " << match[1] << std::endl;
        std::cout << "月: " << match[2] << std::endl;
        std::cout << "日: " << match[3] << std::endl;
    }

    return 0;
}
```

### 命名捕获组

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Name: John, Age: 30";

    // 命名捕获组
    boost::regex pattern(R"(Name: (?<name>\w+), Age: (?<age>\d+))");
    boost::smatch match;

    if (boost::regex_search(text, match, pattern)) {
        std::cout << "姓名: " << match["name"] << std::endl;
        std::cout << "年龄: " << match["age"] << std::endl;
    }

    return 0;
}
```

---

## 查找所有匹配

### regex_iterator

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "电话：138-1234-5678, 139-5678-1234, 150-9999-8888";

    boost::regex phone_pattern(R"(\d{3}-\d{4}-\d{4})");

    // 迭代所有匹配
    boost::sregex_iterator it(text.begin(), text.end(), phone_pattern);
    boost::sregex_iterator end;

    int count = 0;
    for (; it != end; ++it) {
        std::cout << "电话 " << ++count << ": " << it->str() << std::endl;
    }

    return 0;
}
```

### regex_token_iterator

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "apple,banana,orange,grape";

    // 使用逗号作为分隔符
    boost::regex sep_pattern(R"(,)");

    boost::sregex_token_iterator it(text.begin(), text.end(), sep_pattern, -1);
    boost::sregex_token_iterator end;

    std::cout << "水果列表:\n";
    for (; it != end; ++it) {
        std::cout << "- " << *it << std::endl;
    }

    return 0;
}
```

---

## 替换

### regex_replace - 简单替换

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "电话：138-1234-5678";

    // 隐藏中间4位
    boost::regex pattern(R"(\d{3}-(\d{4})-\d{4})");
    std::string result = boost::regex_replace(
        text,
        pattern,
        R"(***-$1-****)"
    );

    std::cout << "原文: " << text << std::endl;
    std::cout << "替换后: " << result << std::endl;

    return 0;
}
```

### 使用回调函数替换

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

std::string to_upper(const boost::smatch& match) {
    std::string word = match[0].str();
    for (char& c : word) {
        c = std::toupper(c);
    }
    return word;
}

int main() {
    std::string text = "hello world, boost regex";

    boost::regex word_pattern(R"(\w+)");

    // 将所有单词转为大写
    std::string result = boost::regex_replace(
        text,
        word_pattern,
        to_upper,
        boost::match_default | boost::format_all
    );

    std::cout << "原文: " << text << std::endl;
    std::cout << "替换后: " << result << std::endl;

    return 0;
}
```

---

## 正则表达式语法

### 基本元字符

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <vector>

void test_pattern(const std::string& pattern_str, const std::vector<std::string>& tests) {
    boost::regex pattern(pattern_str);

    std::cout << "模式: " << pattern_str << std::endl;
    for (const auto& test : tests) {
        bool match = boost::regex_match(test, pattern);
        std::cout << "  \"" << test << "\" -> " << (match ? "✓" : "✗") << std::endl;
    }
    std::cout << std::endl;
}

int main() {
    // 1. 字符类
    test_pattern(R"([abc]+)", {"abc", "aaa", "bca", "xyz"});

    // 2. 数字
    test_pattern(R"(\d{3})", {"123", "abc", "12"});

    // 3. 单词字符
    test_pattern(R"(\w+)", {"hello", "123", "hello_123", "你好"});

    // 4. 空白字符
    test_pattern(R"(\s+)", {" ", "\t", "\n", "abc"});

    // 5. 任意字符
    test_pattern(R"(.{3})", {"abc", "123", "a\nb", "ab"});

    // 6. 锚点
    test_pattern(R"(^hello$)", {"hello", "hello world", "say hello"});

    return 0;
}
```

### 量词

```cpp
#include <boost/regex.hpp>
#include <iostream>

int main() {
    std::vector<std::pair<std::string, std::string>> tests = {
        {R"(a?)", "a, ''"},              // 0 或 1 次
        {R"(a*)", "aaa, ''"},            // 0 或多次
        {R"(a+)", "aaa"},                // 1 或多次
        {R"(a{3})", "aaa"},              // 恰好 3 次
        {R"(a{2,4})", "aa, aaa, aaaa"},  // 2 到 4 次
        {R"(a{2,})", "aa, aaa, aaaa"},   // 至少 2 次
    };

    for (const auto& [pattern, description] : tests) {
        std::cout << pattern << " -> " << description << std::endl;
    }

    return 0;
}
```

### 贪婪与非贪婪

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string html = "<div>content</div>";

    // 贪婪匹配
    boost::regex greedy(R"(<.*>)");
    boost::smatch match1;
    if (boost::regex_search(html, match1, greedy)) {
        std::cout << "贪婪: " << match1[0] << std::endl;  // <div>content</div>
    }

    // 非贪婪匹配
    boost::regex non_greedy(R"(<.*?>)");
    boost::smatch match2;
    if (boost::regex_search(html, match2, non_greedy)) {
        std::cout << "非贪婪: " << match2[0] << std::endl;  // <div>
    }

    return 0;
}
```

---

## 实用示例

### 邮箱验证

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>
#include <vector>

bool is_valid_email(const std::string& email) {
    boost::regex pattern(
        R"(^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$)"
    );
    return boost::regex_match(email, pattern);
}

int main() {
    std::vector<std::string> emails = {
        "user@example.com",
        "test.user@domain.co.uk",
        "invalid@",
        "@invalid.com",
        "no-at-sign.com"
    };

    for (const auto& email : emails) {
        std::cout << email << " -> "
                  << (is_valid_email(email) ? "有效" : "无效")
                  << std::endl;
    }

    return 0;
}
```

### URL 解析

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

struct URL {
    std::string protocol;
    std::string host;
    std::string port;
    std::string path;
    std::string query;
};

URL parse_url(const std::string& url_str) {
    boost::regex pattern(
        R"(^(https?):\/\/([^:\/]+)(?::(\d+))?(\/[^?]*)?(?:\?(.*))?$)"
    );

    boost::smatch match;
    URL url;

    if (boost::regex_match(url_str, match, pattern)) {
        url.protocol = match[1];
        url.host = match[2];
        url.port = match[3];
        url.path = match[4];
        url.query = match[5];
    }

    return url;
}

int main() {
    std::string url_str = "https://www.example.com:8080/path/to/page?key=value&foo=bar";

    URL url = parse_url(url_str);

    std::cout << "协议: " << url.protocol << std::endl;
    std::cout << "主机: " << url.host << std::endl;
    std::cout << "端口: " << url.port << std::endl;
    std::cout << "路径: " << url.path << std::endl;
    std::cout << "查询: " << url.query << std::endl;

    return 0;
}
```

### 日志解析

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>
#include <fstream>

struct LogEntry {
    std::string timestamp;
    std::string level;
    std::string message;
};

LogEntry parse_log_line(const std::string& line) {
    // [2024-03-15 10:30:45] [INFO] Application started
    boost::regex pattern(
        R"(\[([^\]]+)\]\s+\[(\w+)\]\s+(.+))"
    );

    boost::smatch match;
    LogEntry entry;

    if (boost::regex_match(line, match, pattern)) {
        entry.timestamp = match[1];
        entry.level = match[2];
        entry.message = match[3];
    }

    return entry;
}

int main() {
    std::string log_line = "[2024-03-15 10:30:45] [INFO] Application started";

    LogEntry entry = parse_log_line(log_line);

    std::cout << "时间: " << entry.timestamp << std::endl;
    std::cout << "级别: " << entry.level << std::endl;
    std::cout << "消息: " << entry.message << std::endl;

    return 0;
}
```

### 密码强度验证

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

class PasswordValidator {
public:
    static bool validate(const std::string& password, std::string& error) {
        // 至少 8 个字符
        if (password.length() < 8) {
            error = "密码长度至少 8 个字符";
            return false;
        }

        // 包含大写字母
        if (!boost::regex_search(password, boost::regex(R"([A-Z])"))) {
            error = "密码必须包含大写字母";
            return false;
        }

        // 包含小写字母
        if (!boost::regex_search(password, boost::regex(R"([a-z])"))) {
            error = "密码必须包含小写字母";
            return false;
        }

        // 包含数字
        if (!boost::regex_search(password, boost::regex(R"(\d)"))) {
            error = "密码必须包含数字";
            return false;
        }

        // 包含特殊字符
        if (!boost::regex_search(password, boost::regex(R"([@#$%^&*()_+\-=\[\]{}|;:,.<>?])"))) {
            error = "密码必须包含特殊字符";
            return false;
        }

        return true;
    }
};

int main() {
    std::vector<std::string> passwords = {
        "weak",
        "NoSpecial123",
        "no_upper123!",
        "NO_LOWER123!",
        "NoDigits!",
        "StrongP@ss123"
    };

    for (const auto& pwd : passwords) {
        std::string error;
        bool valid = PasswordValidator::validate(pwd, error);

        std::cout << "\"" << pwd << "\" -> ";
        if (valid) {
            std::cout << "✓ 有效" << std::endl;
        } else {
            std::cout << "✗ " << error << std::endl;
        }
    }

    return 0;
}
```

---

## 性能优化

### 预编译正则表达式

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <chrono>

int main() {
    std::vector<std::string> emails(10000, "user@example.com");

    // 预编译（推荐）
    auto start1 = std::chrono::high_resolution_clock::now();
    boost::regex pattern(R"(\w+@\w+\.\w+)");
    for (const auto& email : emails) {
        boost::regex_match(email, pattern);
    }
    auto end1 = std::chrono::high_resolution_clock::now();

    // 每次编译（不推荐）
    auto start2 = std::chrono::high_resolution_clock::now();
    for (const auto& email : emails) {
        boost::regex temp_pattern(R"(\w+@\w+\.\w+)");
        boost::regex_match(email, temp_pattern);
    }
    auto end2 = std::chrono::high_resolution_clock::now();

    auto time1 = std::chrono::duration_cast<std::chrono::milliseconds>(end1 - start1).count();
    auto time2 = std::chrono::duration_cast<std::chrono::milliseconds>(end2 - start2).count();

    std::cout << "预编译: " << time1 << " ms" << std::endl;
    std::cout << "每次编译: " << time2 << " ms" << std::endl;

    return 0;
}
```

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_regex -o example

# CMake
find_package(Boost REQUIRED COMPONENTS regex)
target_link_libraries(myapp Boost::regex)
```

---

## 参考资源

- [Boost.Regex 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/regex/doc/html/index.html)
- [正则表达式语法参考](https://www.boost.org/doc/libs/1_90_0/libs/regex/doc/html/boost_regex/syntax.html)

# Boost.Regex - 正则表达式库

## 概述

Boost.Regex 提供正则表达式支持，是 C++11 std::regex 的前身。

**类型**: 需要编译的库

**注意**: C++11 引入了 std::regex，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "My email is user@example.com";
    boost::regex pattern(R"(\w+@\w+\.\w+)");

    boost::smatch match;
    if (boost::regex_search(text, match, pattern)) {
        std::cout << "找到邮箱: " << match[0] << std::endl;
    }

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_regex`

---

## 匹配检查

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    boost::regex pattern(R"(\d{3}-\d{4}-\d{4})");  // 电话号码格式

    std::vector<std::string> phones = {
        "010-1234-5678",
        "123-4567-8901",
        "12-345-6789",  // 不匹配
        "010-1234-567"  // 不匹配
    };

    for (const auto& phone : phones) {
        if (boost::regex_match(phone, pattern)) {
            std::cout << phone << " ✓" << std::endl;
        } else {
            std::cout << phone << " ✗" << std::endl;
        }
    }

    return 0;
}
```

---

## 搜索匹配

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "联系方式: user@example.com 或 admin@test.org";
    boost::regex pattern(R"(\w+@\w+\.\w+)");

    boost::smatch match;
    std::string::const_iterator start = text.begin();
    std::string::const_iterator end = text.end();

    std::cout << "找到的邮箱:\n";
    while (boost::regex_search(start, end, match, pattern)) {
        std::cout << "  " << match[0] << std::endl;
        start = match[0].second;
    }

    return 0;
}
```

---

## 捕获组

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "日期: 2024-11-18";
    boost::regex pattern(R"((\d{4})-(\d{2})-(\d{2}))");

    boost::smatch match;
    if (boost::regex_search(text, match, pattern)) {
        std::cout << "完整匹配: " << match[0] << std::endl;
        std::cout << "年: " << match[1] << std::endl;
        std::cout << "月: " << match[2] << std::endl;
        std::cout << "日: " << match[3] << std::endl;
    }

    return 0;
}
```

---

## 替换操作

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Hello World! Hello Boost!";

    // 简单替换
    boost::regex pattern1("Hello");
    std::string result1 = boost::regex_replace(text, pattern1, "Hi");
    std::cout << "替换后: " << result1 << std::endl;

    // 使用捕获组
    std::string date = "2024-11-18";
    boost::regex pattern2(R"((\d{4})-(\d{2})-(\d{2}))");
    std::string result2 = boost::regex_replace(date, pattern2, "$3/$2/$1");
    std::cout << "日期转换: " << result2 << std::endl;

    return 0;
}
```

---

## 分割字符串

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "apple,banana;orange:grape";
    boost::regex pattern("[,;:]");

    boost::sregex_token_iterator it(text.begin(), text.end(), pattern, -1);
    boost::sregex_token_iterator end;

    std::cout << "分割结果:\n";
    while (it != end) {
        std::cout << "  " << *it++ << std::endl;
    }

    return 0;
}
```

---

## 邮箱验证

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
        std::cout << email << ": " 
                  << (is_valid_email(email) ? "有效" : "无效") 
                  << std::endl;
    }

    return 0;
}
```

---

## URL 解析

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

struct URL {
    std::string protocol;
    std::string host;
    std::string port;
    std::string path;
};

URL parse_url(const std::string& url) {
    boost::regex pattern(
        R"(^(https?):\/\/([^:\/\s]+)(?::(\d+))?(\/[^\s]*)?$)"
    );

    boost::smatch match;
    URL result;

    if (boost::regex_match(url, match, pattern)) {
        result.protocol = match[1];
        result.host = match[2];
        result.port = match[3].matched ? match[3].str() : "80";
        result.path = match[4].matched ? match[4].str() : "/";
    }

    return result;
}

int main() {
    std::string url = "https://www.example.com:8080/path/to/page";
    URL parsed = parse_url(url);

    std::cout << "协议: " << parsed.protocol << std::endl;
    std::cout << "主机: " << parsed.host << std::endl;
    std::cout << "端口: " << parsed.port << std::endl;
    std::cout << "路径: " << parsed.path << std::endl;

    return 0;
}
```

---

## 大小写不敏感匹配

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "Hello WORLD hello";

    // 大小写敏感（默认）
    boost::regex pattern1("hello");
    std::cout << "大小写敏感匹配: " 
              << boost::regex_search(text, pattern1) << std::endl;

    // 大小写不敏感
    boost::regex pattern2("hello", boost::regex::icase);
    
    boost::smatch match;
    std::string::const_iterator start = text.begin();
    
    std::cout << "大小写不敏感匹配:\n";
    while (boost::regex_search(start, text.end(), match, pattern2)) {
        std::cout << "  " << match[0] << std::endl;
        start = match[0].second;
    }

    return 0;
}
```

---

## HTML 标签移除

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

std::string remove_html_tags(const std::string& html) {
    boost::regex pattern("<[^>]*>");
    return boost::regex_replace(html, pattern, "");
}

int main() {
    std::string html = "<p>This is <b>bold</b> and <i>italic</i> text.</p>";

    std::cout << "原始: " << html << std::endl;
    std::cout << "清理后: " << remove_html_tags(html) << std::endl;

    return 0;
}
```

---

## 电话号码格式化

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

std::string format_phone(const std::string& phone) {
    // 移除所有非数字字符
    boost::regex non_digit(R"(\D)");
    std::string digits = boost::regex_replace(phone, non_digit, "");

    // 格式化为 XXX-XXXX-XXXX
    if (digits.length() == 11) {
        boost::regex pattern(R"((\d{3})(\d{4})(\d{4}))");
        return boost::regex_replace(digits, pattern, "$1-$2-$3");
    }

    return phone;
}

int main() {
    std::vector<std::string> phones = {
        "13812345678",
        "138 1234 5678",
        "138-1234-5678",
        "(138) 1234-5678"
    };

    for (const auto& phone : phones) {
        std::cout << phone << " -> " << format_phone(phone) << std::endl;
    }

    return 0;
}
```

---

## 密码强度检查

```cpp
#include <boost/regex.hpp>
#include <iostream>
#include <string>

bool is_strong_password(const std::string& password) {
    // 至少8个字符，包含大小写字母、数字和特殊字符
    if (password.length() < 8) return false;

    boost::regex has_lower("[a-z]");
    boost::regex has_upper("[A-Z]");
    boost::regex has_digit("[0-9]");
    boost::regex has_special("[!@#$%^&*]");

    return boost::regex_search(password, has_lower) &&
           boost::regex_search(password, has_upper) &&
           boost::regex_search(password, has_digit) &&
           boost::regex_search(password, has_special);
}

int main() {
    std::vector<std::string> passwords = {
        "weak",
        "WeakPass",
        "Weak123",
        "Strong123!",
        "VeryStr0ng!"
    };

    for (const auto& pwd : passwords) {
        std::cout << pwd << ": " 
                  << (is_strong_password(pwd) ? "强" : "弱") 
                  << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Regex 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/regex/doc/html/index.html)
- [C++11 std::regex](https://en.cppreference.com/w/cpp/regex)

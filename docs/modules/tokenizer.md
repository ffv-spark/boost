# Boost.Tokenizer - 字符串分词库

## 概述

Boost.Tokenizer 提供灵活的字符串分词功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "one,two,three,four";

    boost::tokenizer<> tok(text);

    for (const auto& token : tok) {
        std::cout << token << std::endl;
    }

    return 0;
}
```

---

## 字符分隔符

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "apple,banana;orange|grape";

    // 使用多个分隔符
    boost::char_separator<char> sep(",;|");
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    std::cout << "Tokens:\n";
    for (const auto& token : tok) {
        std::cout << "  - " << token << std::endl;
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

int main() {
    std::string csv = "\"John Doe\",25,\"New York\"";

    boost::escaped_list_separator<char> sep('\\', ',', '\"');
    boost::tokenizer<boost::escaped_list_separator<char>> tok(csv, sep);

    std::cout << "CSV fields:\n";
    for (const auto& field : tok) {
        std::cout << "  [" << field << "]" << std::endl;
    }

    return 0;
}
```

---

## 空格分词

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "  one   two  three   ";

    // 分隔符是空格，丢弃空 token
    boost::char_separator<char> sep(" ", "", boost::drop_empty_tokens);
    boost::tokenizer<boost::char_separator<char>> tok(text, sep);

    for (const auto& token : tok) {
        std::cout << "[" << token << "]" << std::endl;
    }

    return 0;
}
```

---

## 自定义分词器

```cpp
#include <boost/tokenizer.hpp>
#include <iostream>
#include <string>

// 按数字分词
struct digit_separator {
    void reset() {}

    template<typename InputIterator, typename Token>
    bool operator()(InputIterator& next, InputIterator end, Token& tok) {
        tok.clear();

        // 跳过非数字
        while (next != end && !isdigit(*next)) ++next;

        // 收集数字
        while (next != end && isdigit(*next)) {
            tok += *next;
            ++next;
        }

        return !tok.empty();
    }
};

int main() {
    std::string text = "abc123def456ghi789";

    boost::tokenizer<digit_separator> tok(text);

    std::cout << "Numbers:\n";
    for (const auto& token : tok) {
        std::cout << "  " << token << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Tokenizer 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/tokenizer/doc/index.html)

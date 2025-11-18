# Boost.Spirit - 解析器生成器

## 概述

Boost.Spirit 是一个基于 C++ 模板的解析器生成器框架，可以直接在 C++ 代码中定义语法规则。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

int main() {
    std::string input = "42";
    int value;

    // 解析整数
    bool success = qi::parse(
        input.begin(), input.end(),
        qi::int_,
        value
    );

    if (success) {
        std::cout << "解析成功: " << value << std::endl;
    } else {
        std::cout << "解析失败" << std::endl;
    }

    return 0;
}
```

---

## 解析数字

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

int main() {
    std::string input = "123 45.67 0x1A";
    auto it = input.begin();

    int i;
    double d;
    int hex;

    // 解析整数
    qi::parse(it, input.end(), qi::int_, i);
    std::cout << "整数: " << i << std::endl;

    // 跳过空格
    qi::phrase_parse(it, input.end(), qi::double_, qi::space, d);
    std::cout << "浮点数: " << d << std::endl;

    // 解析十六进制
    qi::phrase_parse(it, input.end(), qi::hex, qi::space, hex);
    std::cout << "十六进制: " << hex << " (十进制: " << hex << ")" << std::endl;

    return 0;
}
```

---

## 解析字符串

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

int main() {
    std::string input = "\"Hello, World!\"";
    std::string result;

    // 解析带引号的字符串
    bool success = qi::parse(
        input.begin(), input.end(),
        '"' >> *(qi::char_ - '"') >> '"',
        result
    );

    if (success) {
        std::cout << "解析的字符串: " << result << std::endl;
    }

    return 0;
}
```

---

## 解析列表

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <vector>
#include <string>

namespace qi = boost::spirit::qi;

int main() {
    std::string input = "1, 2, 3, 4, 5";
    std::vector<int> numbers;

    // 解析逗号分隔的整数列表
    bool success = qi::phrase_parse(
        input.begin(), input.end(),
        qi::int_ % ',',  // % 表示分隔符
        qi::space,
        numbers
    );

    if (success) {
        std::cout << "解析的数字: ";
        for (int n : numbers) {
            std::cout << n << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 自定义规则

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

template <typename Iterator>
struct phone_parser : qi::grammar<Iterator, std::string()> {
    phone_parser() : phone_parser::base_type(phone) {
        // 电话号码格式: XXX-XXXX-XXXX
        phone = qi::repeat(3)[qi::digit] >> '-'
             >> qi::repeat(4)[qi::digit] >> '-'
             >> qi::repeat(4)[qi::digit];
    }

    qi::rule<Iterator, std::string()> phone;
};

int main() {
    std::string input = "010-1234-5678";
    std::string result;

    phone_parser<std::string::iterator> parser;

    bool success = qi::parse(
        input.begin(), input.end(),
        parser,
        result
    );

    if (success) {
        std::cout << "有效的电话号码: " << result << std::endl;
    } else {
        std::cout << "无效的电话号码" << std::endl;
    }

    return 0;
}
```

---

## 解析JSON

```cpp
#include <boost/spirit/include/qi.hpp>
#include <boost/variant.hpp>
#include <boost/fusion/include/adapt_struct.hpp>
#include <iostream>
#include <string>
#include <map>

namespace qi = boost::spirit::qi;

// 简化的JSON值
typedef boost::variant<
    std::string,
    double,
    bool
> json_value;

template <typename Iterator>
struct simple_json_parser : qi::grammar<Iterator, json_value(), qi::space_type> {
    simple_json_parser() : simple_json_parser::base_type(value) {
        string_value = '"' >> *(qi::char_ - '"') >> '"';
        number_value = qi::double_;
        bool_value = qi::bool_;
        
        value = string_value | number_value | bool_value;
    }

    qi::rule<Iterator, std::string(), qi::space_type> string_value;
    qi::rule<Iterator, double(), qi::space_type> number_value;
    qi::rule<Iterator, bool(), qi::space_type> bool_value;
    qi::rule<Iterator, json_value(), qi::space_type> value;
};

int main() {
    std::vector<std::string> inputs = {
        "\"hello\"",
        "42.5",
        "true"
    };

    simple_json_parser<std::string::iterator> parser;

    for (const auto& input : inputs) {
        json_value result;
        auto it = input.begin();

        bool success = qi::phrase_parse(
            it, input.end(),
            parser,
            qi::space,
            result
        );

        if (success) {
            std::cout << "解析成功: " << input << std::endl;
        }
    }

    return 0;
}
```

---

## 解析表达式

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

template <typename Iterator>
struct calculator : qi::grammar<Iterator, int(), qi::space_type> {
    calculator() : calculator::base_type(expression) {
        expression = term >> *(('+' >> term) | ('-' >> term));
        term = factor >> *(('*' >> factor) | ('/' >> factor));
        factor = qi::int_ | ('(' >> expression >> ')');
    }

    qi::rule<Iterator, int(), qi::space_type> expression, term, factor;
};

int main() {
    std::string input = "2 + 3 * 4";

    calculator<std::string::iterator> calc;
    int result;

    bool success = qi::phrase_parse(
        input.begin(), input.end(),
        calc,
        qi::space,
        result
    );

    if (success) {
        std::cout << input << " = " << result << std::endl;
    }

    return 0;
}
```

---

## Karma 生成器

```cpp
#include <boost/spirit/include/karma.hpp>
#include <iostream>
#include <string>
#include <vector>

namespace karma = boost::spirit::karma;

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::string output;

    // 生成逗号分隔的列表
    karma::generate(
        std::back_inserter(output),
        karma::int_ % ", ",
        numbers
    );

    std::cout << "生成的字符串: " << output << std::endl;

    return 0;
}
```

---

## 解析配置文件

```cpp
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/include/adapt_struct.hpp>
#include <iostream>
#include <string>
#include <map>

namespace qi = boost::spirit::qi;

struct config_entry {
    std::string key;
    std::string value;
};

BOOST_FUSION_ADAPT_STRUCT(
    config_entry,
    (std::string, key)
    (std::string, value)
)

template <typename Iterator>
struct config_parser : qi::grammar<Iterator, std::vector<config_entry>(), qi::space_type> {
    config_parser() : config_parser::base_type(config) {
        key = qi::char_("a-zA-Z_") >> *qi::char_("a-zA-Z0-9_");
        value = *(qi::char_ - qi::eol);
        entry = key >> '=' >> value;
        config = *entry;
    }

    qi::rule<Iterator, std::string()> key, value;
    qi::rule<Iterator, config_entry(), qi::space_type> entry;
    qi::rule<Iterator, std::vector<config_entry>(), qi::space_type> config;
};

int main() {
    std::string input = 
        "name = MyApp\n"
        "version = 1.0\n"
        "port = 8080\n";

    config_parser<std::string::iterator> parser;
    std::vector<config_entry> entries;

    bool success = qi::phrase_parse(
        input.begin(), input.end(),
        parser,
        qi::space,
        entries
    );

    if (success) {
        std::cout << "配置项:\\n";
        for (const auto& entry : entries) {
            std::cout << "  " << entry.key << " = " << entry.value << std::endl;
        }
    }

    return 0;
}
```

---

## 解析CSV

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <vector>
#include <string>

namespace qi = boost::spirit::qi;

template <typename Iterator>
struct csv_parser : qi::grammar<Iterator, std::vector<std::vector<std::string>>()> {
    csv_parser() : csv_parser::base_type(csv) {
        field = '"' >> *(qi::char_ - '"') >> '"'
              | *(qi::char_ - ',' - qi::eol);
        
        record = field % ',';
        csv = record % qi::eol;
    }

    qi::rule<Iterator, std::string()> field;
    qi::rule<Iterator, std::vector<std::string>()> record;
    qi::rule<Iterator, std::vector<std::vector<std::string>>()> csv;
};

int main() {
    std::string input = 
        "Name,Age,City\n"
        "Alice,30,Beijing\n"
        "Bob,25,Shanghai\n"
        "\"Charlie Chan\",35,\"Hong Kong\"";

    csv_parser<std::string::iterator> parser;
    std::vector<std::vector<std::string>> data;

    bool success = qi::parse(
        input.begin(), input.end(),
        parser,
        data
    );

    if (success) {
        std::cout << "CSV 数据:\\n";
        for (const auto& row : data) {
            for (size_t i = 0; i < row.size(); ++i) {
                std::cout << row[i];
                if (i < row.size() - 1) std::cout << " | ";
            }
            std::cout << std::endl;
        }
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Spirit 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/index.html)
- [Spirit Qi 教程](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/spirit/qi.html)
- [Spirit Karma 教程](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/spirit/karma.html)

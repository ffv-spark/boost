# Boost.Spirit - 解析器框架

## 概述

Boost.Spirit 是一个强大的解析器和生成器框架，使用 C++ 模板元编程技术，允许直接在 C++ 代码中编写 EBNF 风格的语法规则。

**类型**: 仅头文件库

**注意**: Spirit 是一个高级库，学习曲线较陡峭，但功能强大

---

## 快速开始 - 简单数字解析

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

int main() {
    std::string input = "42";

    int value;
    auto iter = input.begin();
    auto end = input.end();

    // 解析整数
    bool success = qi::parse(iter, end, qi::int_, value);

    if (success && iter == end) {
        std::cout << "Parsed value: " << value << std::endl;
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## 解析多个值

```cpp
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/include/adapt_struct.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;

struct Point {
    int x;
    int y;
};

// 适配结构体以便 Spirit 使用
BOOST_FUSION_ADAPT_STRUCT(
    Point,
    (int, x)
    (int, y)
)

int main() {
    std::string input = "10, 20";

    Point point;
    auto iter = input.begin();
    auto end = input.end();

    // 语法规则: 整数, 逗号, 空格, 整数
    bool success = qi::phrase_parse(
        iter, end,
        qi::int_ >> ',' >> qi::int_,
        qi::space,  // 跳过空格
        point
    );

    if (success && iter == end) {
        std::cout << "Point: (" << point.x << ", " << point.y << ")" << std::endl;
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## 自定义语法规则

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>
#include <vector>

namespace qi = boost::spirit::qi;
namespace ascii = boost::spirit::ascii;

// 定义语法规则
template <typename Iterator>
struct NumberListGrammar : qi::grammar<Iterator, std::vector<int>(), ascii::space_type> {
    NumberListGrammar() : NumberListGrammar::base_type(start) {
        // 规则: 一个或多个整数，用逗号分隔
        start = qi::int_ % ',';
    }

    qi::rule<Iterator, std::vector<int>(), ascii::space_type> start;
};

int main() {
    std::string input = "1, 2, 3, 4, 5";

    std::vector<int> numbers;
    NumberListGrammar<std::string::iterator> grammar;

    auto iter = input.begin();
    auto end = input.end();

    bool success = qi::phrase_parse(
        iter, end,
        grammar,
        ascii::space,
        numbers
    );

    if (success && iter == end) {
        std::cout << "Parsed numbers: ";
        for (int n : numbers) {
            std::cout << n << " ";
        }
        std::cout << std::endl;
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## JSON 解析示例

```cpp
#include <boost/spirit/include/qi.hpp>
#include <boost/fusion/include/adapt_struct.hpp>
#include <boost/variant.hpp>
#include <iostream>
#include <string>
#include <map>
#include <vector>

namespace qi = boost::spirit::qi;
namespace ascii = boost::spirit::ascii;

// 简化的 JSON 值类型
typedef boost::variant<
    std::string,
    double,
    bool
> JsonValue;

struct JsonObject {
    std::map<std::string, JsonValue> members;
};

BOOST_FUSION_ADAPT_STRUCT(
    JsonObject,
    (std::map<std::string, JsonValue>, members)
)

template <typename Iterator>
struct SimpleJsonGrammar : qi::grammar<Iterator, JsonObject(), ascii::space_type> {
    SimpleJsonGrammar() : SimpleJsonGrammar::base_type(object) {
        using qi::lit;
        using qi::lexeme;
        using ascii::char_;

        // 字符串: "..."
        quoted_string = lexeme['"' >> +(char_ - '"') >> '"'];

        // JSON 值: 字符串、数字或布尔值
        value = quoted_string | qi::double_ | qi::bool_;

        // 键值对: "key": value
        pair = quoted_string >> ':' >> value;

        // 对象: { "key1": value1, "key2": value2 }
        object = '{' >> -(pair % ',') >> '}';
    }

    qi::rule<Iterator, std::string(), ascii::space_type> quoted_string;
    qi::rule<Iterator, JsonValue(), ascii::space_type> value;
    qi::rule<Iterator, std::pair<std::string, JsonValue>(), ascii::space_type> pair;
    qi::rule<Iterator, JsonObject(), ascii::space_type> object;
};

int main() {
    std::string input = R"({
        "name": "John",
        "age": 30,
        "active": true
    })";

    JsonObject obj;
    SimpleJsonGrammar<std::string::iterator> grammar;

    auto iter = input.begin();
    auto end = input.end();

    bool success = qi::phrase_parse(
        iter, end,
        grammar,
        ascii::space,
        obj
    );

    if (success && iter == end) {
        std::cout << "Parsed JSON object with "
                  << obj.members.size() << " members" << std::endl;
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## 计算器示例

```cpp
#include <boost/spirit/include/qi.hpp>
#include <boost/spirit/include/phoenix_operator.hpp>
#include <iostream>
#include <string>

namespace qi = boost::spirit::qi;
namespace ascii = boost::spirit::ascii;

template <typename Iterator>
struct CalculatorGrammar : qi::grammar<Iterator, int(), ascii::space_type> {
    CalculatorGrammar() : CalculatorGrammar::base_type(expression) {
        using qi::_val;
        using qi::_1;
        using qi::int_;

        // 表达式 = 项 + 项 - 项
        expression = term[_val = _1]
                   >> *(('+' >> term[_val += _1])
                   |    ('-' >> term[_val -= _1]));

        // 项 = 因子 * 因子 / 因子
        term = factor[_val = _1]
             >> *(('*' >> factor[_val *= _1])
             |    ('/' >> factor[_val /= _1]));

        // 因子 = 数字 或 (表达式)
        factor = int_[_val = _1]
               | ('(' >> expression[_val = _1] >> ')');
    }

    qi::rule<Iterator, int(), ascii::space_type> expression;
    qi::rule<Iterator, int(), ascii::space_type> term;
    qi::rule<Iterator, int(), ascii::space_type> factor;
};

int main() {
    std::string input = "2 * (3 + 4) - 5";

    int result;
    CalculatorGrammar<std::string::iterator> calc;

    auto iter = input.begin();
    auto end = input.end();

    bool success = qi::phrase_parse(
        iter, end,
        calc,
        ascii::space,
        result
    );

    if (success && iter == end) {
        std::cout << input << " = " << result << std::endl;
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## CSV 解析

```cpp
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>
#include <vector>

namespace qi = boost::spirit::qi;

template <typename Iterator>
bool parse_csv_line(Iterator first, Iterator last, std::vector<std::string>& v) {
    using qi::lexeme;
    using qi::char_;
    using qi::_1;

    // CSV 字段规则
    auto field = lexeme[+(char_ - ',')];

    // 整行规则: 字段 % 逗号
    bool r = qi::parse(first, last, field % ',', v);

    if (first != last) {
        return false;
    }

    return r;
}

int main() {
    std::string line = "Alice,30,New York";
    std::vector<std::string> fields;

    if (parse_csv_line(line.begin(), line.end(), fields)) {
        std::cout << "Parsed " << fields.size() << " fields:\n";
        for (const auto& field : fields) {
            std::cout << "  [" << field << "]" << std::endl;
        }
    } else {
        std::cout << "Parsing failed" << std::endl;
    }

    return 0;
}
```

---

## Karma - 生成器（与 Qi 相反）

```cpp
#include <boost/spirit/include/karma.hpp>
#include <iostream>
#include <string>
#include <vector>

namespace karma = boost::spirit::karma;

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    std::string output;
    auto iter = std::back_inserter(output);

    // 生成逗号分隔的数字列表
    bool success = karma::generate(
        iter,
        karma::int_ % ", ",  // 格式: 整数，用 ", " 分隔
        numbers
    );

    if (success) {
        std::cout << "Generated: " << output << std::endl;
    }

    // 生成更复杂的格式
    output.clear();
    iter = std::back_inserter(output);

    karma::generate(
        iter,
        '[' << (karma::int_ % ", ") << ']',
        numbers
    );

    std::cout << "Generated with brackets: " << output << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Spirit 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/index.html)
- [Spirit Qi 教程](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/spirit/qi/tutorials.html)
- [Spirit Karma 文档](https://www.boost.org/doc/libs/1_90_0/libs/spirit/doc/html/spirit/karma.html)

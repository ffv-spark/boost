# Boost.StringAlgo - 字符串算法库

## 概述

Boost.StringAlgo 提供丰富的字符串处理算法，包括大小写转换、分割、查找替换等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World";

    // 转换为大写
    std::string upper = boost::to_upper_copy(str);
    std::cout << "大写: " << upper << std::endl;

    // 转换为小写
    std::string lower = boost::to_lower_copy(str);
    std::cout << "小写: " << lower << std::endl;

    // 原地转换
    boost::to_upper(str);
    std::cout << "原地大写: " << str << std::endl;

    return 0;
}
```

---

## 去除空格

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "   Hello World   ";

    std::cout << "原始: [" << str << "]" << std::endl;

    // 去除左侧空格
    std::string left = boost::trim_left_copy(str);
    std::cout << "左去除: [" << left << "]" << std::endl;

    // 去除右侧空格
    std::string right = boost::trim_right_copy(str);
    std::cout << "右去除: [" << right << "]" << std::endl;

    // 去除两侧空格
    std::string both = boost::trim_copy(str);
    std::cout << "两侧去除: [" << both << "]" << std::endl;

    // 原地去除
    boost::trim(str);
    std::cout << "原地去除: [" << str << "]" << std::endl;

    return 0;
}
```

---

## 字符串分割

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::string str = "apple,banana,orange,grape";
    std::vector<std::string> parts;

    // 按逗号分割
    boost::split(parts, str, boost::is_any_of(","));

    std::cout << "分割结果:\n";
    for (const auto& part : parts) {
        std::cout << "  [" << part << "]" << std::endl;
    }

    // 分割并去除空格
    std::string str2 = " one , two , three ";
    std::vector<std::string> parts2;
    boost::split(parts2, str2, boost::is_any_of(","));

    std::cout << "\n分割后去空格:\n";
    for (auto& part : parts2) {
        boost::trim(part);
        std::cout << "  [" << part << "]" << std::endl;
    }

    return 0;
}
```

---

## 字符串连接

```cpp
#include <boost/algorithm/string/join.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> words = {"Hello", "Boost", "World"};

    // 用空格连接
    std::string joined = boost::join(words, " ");
    std::cout << "空格连接: " << joined << std::endl;

    // 用逗号连接
    std::string csv = boost::join(words, ", ");
    std::cout << "逗号连接: " << csv << std::endl;

    return 0;
}
```

---

## 查找和替换

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World, Hello Boost";

    // 查找第一个匹配
    if (boost::contains(str, "World")) {
        std::cout << "包含 'World'" << std::endl;
    }

    // 替换第一个匹配
    std::string replaced = boost::replace_first_copy(str, "Hello", "Hi");
    std::cout << "替换第一个: " << replaced << std::endl;

    // 替换所有匹配
    std::string all_replaced = boost::replace_all_copy(str, "Hello", "Hi");
    std::cout << "替换所有: " << all_replaced << std::endl;

    // 原地替换
    boost::replace_all(str, "Hello", "Greetings");
    std::cout << "原地替换: " << str << std::endl;

    return 0;
}
```

---

## 大小写忽略比较

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string s1 = "Hello World";
    std::string s2 = "hello world";
    std::string s3 = "HELLO WORLD";

    // 不区分大小写比较
    std::cout << std::boolalpha;
    std::cout << "s1 == s2 (忽略大小写): "
              << boost::iequals(s1, s2) << std::endl;

    std::cout << "s1 == s3 (忽略大小写): "
              << boost::iequals(s1, s3) << std::endl;

    // 不区分大小写查找
    if (boost::ifind_first(s1, "WORLD")) {
        std::cout << "找到 'WORLD' (忽略大小写)" << std::endl;
    }

    return 0;
}
```

---

## 前缀后缀检查

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string filename = "document.txt";

    // 检查前缀
    if (boost::starts_with(filename, "doc")) {
        std::cout << "以 'doc' 开头" << std::endl;
    }

    // 检查后缀
    if (boost::ends_with(filename, ".txt")) {
        std::cout << "以 '.txt' 结尾" << std::endl;
    }

    // 不区分大小写
    std::string upper_file = "DOCUMENT.TXT";
    if (boost::iends_with(upper_file, ".txt")) {
        std::cout << "忽略大小写也以 '.txt' 结尾" << std::endl;
    }

    return 0;
}
```

---

## 删除子串

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World Hello";

    // 删除第一个匹配
    std::string result1 = boost::erase_first_copy(str, "Hello");
    std::cout << "删除第一个 'Hello': " << result1 << std::endl;

    // 删除最后一个匹配
    std::string result2 = boost::erase_last_copy(str, "Hello");
    std::cout << "删除最后一个 'Hello': " << result2 << std::endl;

    // 删除所有匹配
    std::string result3 = boost::erase_all_copy(str, "Hello");
    std::cout << "删除所有 'Hello': " << result3 << std::endl;

    // 删除指定位置的字符
    std::string result4 = boost::erase_head_copy(str, 6);
    std::cout << "删除前6个字符: " << result4 << std::endl;

    return 0;
}
```

---

## 谓词和分类

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "  Hello123  ";

    // 检查是否全部为空格
    std::cout << std::boolalpha;
    std::cout << "全为空格: " << boost::all(str, boost::is_space()) << std::endl;

    // 修剪非字母数字字符
    std::string trimmed = str;
    boost::trim_if(trimmed, !boost::is_alnum());
    std::cout << "去除非字母数字: [" << trimmed << "]" << std::endl;

    // 按条件分割
    std::string data = "123abc456def789";
    std::vector<std::string> parts;
    boost::split(parts, data, boost::is_digit());

    std::cout << "按数字分割:\n";
    for (const auto& part : parts) {
        if (!part.empty()) {
            std::cout << "  [" << part << "]" << std::endl;
        }
    }

    return 0;
}
```

---

## 查找所有匹配

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "The quick brown fox jumps over the lazy dog";
    std::string pattern = "the";

    // 不区分大小写查找所有匹配
    typedef boost::iterator_range<std::string::iterator> string_range;

    std::string::iterator start = text.begin();
    while (true) {
        string_range result = boost::ifind_first(
            boost::make_iterator_range(start, text.end()),
            pattern
        );

        if (result.empty()) {
            break;
        }

        std::cout << "找到 '" << pattern << "' 在位置 "
                  << (result.begin() - text.begin()) << std::endl;

        start = result.end();
    }

    return 0;
}
```

---

## 字符串格式化

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string text = "hello world";

    // 首字母大写
    std::string capitalized = text;
    if (!capitalized.empty()) {
        capitalized[0] = std::toupper(capitalized[0]);
    }
    std::cout << "首字母大写: " << capitalized << std::endl;

    // 每个单词首字母大写
    std::vector<std::string> words;
    boost::split(words, text, boost::is_space());

    for (auto& word : words) {
        if (!word.empty()) {
            word[0] = std::toupper(word[0]);
        }
    }

    std::string title_case = boost::join(words, " ");
    std::cout << "标题格式: " << title_case << std::endl;

    return 0;
}
```

---

## 性能对比

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>
#include <chrono>

int main() {
    std::string text(1000000, 'a');
    text += "needle";
    text += std::string(1000000, 'b');

    // 使用 Boost
    auto start1 = std::chrono::high_resolution_clock::now();
    bool found1 = boost::contains(text, "needle");
    auto end1 = std::chrono::high_resolution_clock::now();

    // 使用标准库
    auto start2 = std::chrono::high_resolution_clock::now();
    bool found2 = text.find("needle") != std::string::npos;
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration_cast<std::chrono::microseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::microseconds>(end2 - start2);

    std::cout << "Boost.StringAlgo: " << duration1.count() << " 微秒" << std::endl;
    std::cout << "标准库 find: " << duration2.count() << " 微秒" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.StringAlgo 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/string_algo.html)
- [算法参考](https://www.boost.org/doc/libs/1_90_0/doc/html/string_algo/reference.html)

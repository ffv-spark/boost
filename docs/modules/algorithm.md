# Boost.Algorithm - 算法库

## 概述

Boost.Algorithm 提供通用算法扩展，补充 STL 算法库的功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "  Hello World  ";
    
    // 去除空格
    boost::trim(str);
    
    std::cout << "修剪后: '" << str << "'" << std::endl;
    
    // 转大写
    boost::to_upper(str);
    std::cout << "大写: " << str << std::endl;
    
    return 0;
}
```

---

## 字符串修剪

```cpp
#include <boost/algorithm/string/trim.hpp>
#include <iostream>
#include <string>

int main() {
    std::string s1 = "   hello   ";
    std::string s2 = "   world   ";
    std::string s3 = "   boost   ";
    
    // 去除两端空格
    boost::trim(s1);
    std::cout << "trim: '" << s1 << "'" << std::endl;
    
    // 去除左边空格
    boost::trim_left(s2);
    std::cout << "trim_left: '" << s2 << "'" << std::endl;
    
    // 去除右边空格
    boost::trim_right(s3);
    std::cout << "trim_right: '" << s3 << "'" << std::endl;
    
    // 返回修剪后的副本
    std::string s4 = "   copy   ";
    std::string result = boost::trim_copy(s4);
    std::cout << "原始: '" << s4 << "'" << std::endl;
    std::cout << "副本: '" << result << "'" << std::endl;
    
    return 0;
}
```

---

## 大小写转换

```cpp
#include <boost/algorithm/string/case_conv.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World";
    
    // 转大写
    boost::to_upper(str);
    std::cout << "大写: " << str << std::endl;
    
    // 转小写
    boost::to_lower(str);
    std::cout << "小写: " << str << std::endl;
    
    // 返回副本
    std::string original = "BoOsT";
    std::string upper = boost::to_upper_copy(original);
    std::string lower = boost::to_lower_copy(original);
    
    std::cout << "原始: " << original << std::endl;
    std::cout << "大写副本: " << upper << std::endl;
    std::cout << "小写副本: " << lower << std::endl;
    
    return 0;
}
```

---

## 字符串分割

```cpp
#include <boost/algorithm/string/split.hpp>
#include <boost/algorithm/string/classification.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::string str = "apple,banana,orange,grape";
    std::vector<std::string> parts;
    
    // 按逗号分割
    boost::split(parts, str, boost::is_any_of(","));
    
    std::cout << "分割结果:\\n";
    for (const auto& part : parts) {
        std::cout << "  " << part << std::endl;
    }
    
    // 按多个分隔符分割
    std::string str2 = "one;two,three:four";
    std::vector<std::string> parts2;
    boost::split(parts2, str2, boost::is_any_of(";,:"));
    
    std::cout << "\\n多分隔符分割:\\n";
    for (const auto& part : parts2) {
        std::cout << "  " << part << std::endl;
    }
    
    return 0;
}
```

---

## 字符串连接

```cpp
#include <boost/algorithm/string/join.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> words = {"Hello", "World", "from", "Boost"};
    
    // 用空格连接
    std::string sentence = boost::join(words, " ");
    std::cout << "句子: " << sentence << std::endl;
    
    // 用逗号连接
    std::string csv = boost::join(words, ", ");
    std::cout << "CSV: " << csv << std::endl;
    
    return 0;
}
```

---

## 字符串查找

```cpp
#include <boost/algorithm/string/find.hpp>
#include <boost/algorithm/string/predicate.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World, Hello Boost!";
    
    // 前缀检查
    bool starts = boost::starts_with(str, "Hello");
    std::cout << "以 'Hello' 开始: " << starts << std::endl;
    
    // 后缀检查
    bool ends = boost::ends_with(str, "Boost!");
    std::cout << "以 'Boost!' 结束: " << ends << std::endl;
    
    // 包含检查
    bool contains = boost::contains(str, "World");
    std::cout << "包含 'World': " << contains << std::endl;
    
    // 大小写不敏感查找
    bool icontains = boost::icontains(str, "hello");
    std::cout << "包含 'hello' (忽略大小写): " << icontains << std::endl;
    
    return 0;
}
```

---

## 字符串替换

```cpp
#include <boost/algorithm/string/replace.hpp>
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello World, Hello Boost";
    
    // 替换第一个匹配
    boost::replace_first(str, "Hello", "Hi");
    std::cout << "替换第一个: " << str << std::endl;
    
    // 替换所有匹配
    std::string str2 = "Hello World, Hello Boost";
    boost::replace_all(str2, "Hello", "Hi");
    std::cout << "替换所有: " << str2 << std::endl;
    
    // 返回副本
    std::string str3 = "Hello World";
    std::string result = boost::replace_all_copy(str3, "o", "0");
    std::cout << "原始: " << str3 << std::endl;
    std::cout << "替换副本: " << result << std::endl;
    
    return 0;
}
```

---

## all_of / any_of / none_of

```cpp
#include <boost/algorithm/cxx11/all_of.hpp>
#include <boost/algorithm/cxx11/any_of.hpp>
#include <boost/algorithm/cxx11/none_of.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {2, 4, 6, 8, 10};
    
    // 检查是否全部为偶数
    bool all_even = boost::algorithm::all_of(numbers, [](int x) { return x % 2 == 0; });
    std::cout << "全部为偶数: " << all_even << std::endl;
    
    // 检查是否有大于5的
    bool any_gt5 = boost::algorithm::any_of(numbers, [](int x) { return x > 5; });
    std::cout << "有大于5的: " << any_gt5 << std::endl;
    
    // 检查是否没有负数
    bool no_neg = boost::algorithm::none_of(numbers, [](int x) { return x < 0; });
    std::cout << "没有负数: " << no_neg << std::endl;
    
    return 0;
}
```

---

## is_sorted

```cpp
#include <boost/algorithm/cxx11/is_sorted.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> sorted = {1, 2, 3, 4, 5};
    std::vector<int> unsorted = {3, 1, 4, 2, 5};
    
    bool s1 = boost::algorithm::is_sorted(sorted.begin(), sorted.end());
    bool s2 = boost::algorithm::is_sorted(unsorted.begin(), unsorted.end());
    
    std::cout << "sorted 是否有序: " << s1 << std::endl;
    std::cout << "unsorted 是否有序: " << s2 << std::endl;
    
    // 降序检查
    std::vector<int> desc = {5, 4, 3, 2, 1};
    bool s3 = boost::algorithm::is_sorted(desc.begin(), desc.end(), std::greater<int>());
    std::cout << "desc 是否降序: " << s3 << std::endl;
    
    return 0;
}
```

---

## is_palindrome

```cpp
#include <boost/algorithm/cxx11/is_palindrome.hpp>
#include <iostream>
#include <string>

int main() {
    std::string s1 = "radar";
    std::string s2 = "hello";
    std::string s3 = "A man a plan a canal Panama";
    
    bool p1 = boost::algorithm::is_palindrome(s1);
    bool p2 = boost::algorithm::is_palindrome(s2);
    
    std::cout << "'" << s1 << "' 是回文: " << p1 << std::endl;
    std::cout << "'" << s2 << "' 是回文: " << p2 << std::endl;
    
    // 忽略空格和大小写
    auto is_alpha = [](char c) { return std::isalpha(c); };
    auto to_lower = [](char c) { return std::tolower(c); };
    
    std::string filtered;
    for (char c : s3) {
        if (is_alpha(c)) {
            filtered += to_lower(c);
        }
    }
    
    bool p3 = boost::algorithm::is_palindrome(filtered);
    std::cout << "'" << s3 << "' 是回文: " << p3 << std::endl;
    
    return 0;
}
```

---

## clamp

```cpp
#include <boost/algorithm/clamp.hpp>
#include <iostream>

int main() {
    using boost::algorithm::clamp;
    
    // 限制值在范围内
    int val1 = clamp(5, 0, 10);    // 5
    int val2 = clamp(-5, 0, 10);   // 0
    int val3 = clamp(15, 0, 10);   // 10
    
    std::cout << "clamp(5, 0, 10) = " << val1 << std::endl;
    std::cout << "clamp(-5, 0, 10) = " << val2 << std::endl;
    std::cout << "clamp(15, 0, 10) = " << val3 << std::endl;
    
    // 浮点数
    double val4 = clamp(3.14, 0.0, 5.0);
    double val5 = clamp(-1.5, 0.0, 5.0);
    
    std::cout << "clamp(3.14, 0.0, 5.0) = " << val4 << std::endl;
    std::cout << "clamp(-1.5, 0.0, 5.0) = " << val5 << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Algorithm 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/algorithm/doc/html/index.html)
- [String Algorithm](https://www.boost.org/doc/libs/1_90_0/doc/html/string_algo.html)

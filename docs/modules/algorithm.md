# Boost.Algorithm - 算法库

## 概述

Boost.Algorithm 提供了大量实用的算法，扩展了 STL 算法库。

**类型**: 仅头文件库

---

## 字符串算法

```cpp
#include <boost/algorithm/string.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::string str = "  Hello World  ";

    // 去除空格
    boost::trim(str);
    std::cout << "Trim: '" << str << "'" << std::endl;

    // 大小写转换
    boost::to_upper(str);
    std::cout << "Upper: " << str << std::endl;

    boost::to_lower(str);
    std::cout << "Lower: " << str << std::endl;

    // 分割字符串
    std::string text = "one,two,three,four";
    std::vector<std::string> parts;
    boost::split(parts, text, boost::is_any_of(","));

    for (const auto& part : parts) {
        std::cout << "- " << part << std::endl;
    }

    // 连接字符串
    std::string joined = boost::join(parts, " | ");
    std::cout << "Joined: " << joined << std::endl;

    // 替换
    std::string original = "Hello World";
    boost::replace_all(original, "World", "Boost");
    std::cout << "Replace: " << original << std::endl;

    // 判断
    std::string test = "Hello";
    std::cout << "Starts with 'He': "
              << boost::starts_with(test, "He") << std::endl;
    std::cout << "Ends with 'lo': "
              << boost::ends_with(test, "lo") << std::endl;
    std::cout << "Contains 'ell': "
              << boost::contains(test, "ell") << std::endl;

    return 0;
}
```

---

## 查找算法

```cpp
#include <boost/algorithm/cxx11/all_of.hpp>
#include <boost/algorithm/cxx11/any_of.hpp>
#include <boost/algorithm/cxx11/none_of.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums = {2, 4, 6, 8, 10};

    // 全部满足
    bool all_even = boost::algorithm::all_of(nums, [](int n) {
        return n % 2 == 0;
    });
    std::cout << "All even: " << all_even << std::endl;

    // 任一满足
    bool any_gt_5 = boost::algorithm::any_of(nums, [](int n) {
        return n > 5;
    });
    std::cout << "Any > 5: " << any_gt_5 << std::endl;

    // 全不满足
    bool none_odd = boost::algorithm::none_of(nums, [](int n) {
        return n % 2 != 0;
    });
    std::cout << "None odd: " << none_odd << std::endl;

    return 0;
}
```

---

## 范围算法

```cpp
#include <boost/range/algorithm.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> nums = {5, 2, 8, 1, 9, 3};

    // 排序
    boost::sort(nums);

    // 查找
    auto it = boost::find(nums, 8);
    if (it != nums.end()) {
        std::cout << "Found: " << *it << std::endl;
    }

    // 统计
    int count = boost::count(nums, 2);
    std::cout << "Count of 2: " << count << std::endl;

    // 最大最小
    auto max_it = boost::max_element(nums);
    auto min_it = boost::min_element(nums);
    std::cout << "Max: " << *max_it << ", Min: " << *min_it << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Algorithm 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/algorithm/doc/html/index.html)

# Boost.Range - 范围库

## 概述

Boost.Range 提供范围抽象和算法适配器，简化容器和迭代器的操作。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/range/algorithm.hpp>
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 使用范围算法
    auto it = boost::find(numbers, 5);
    if (it != numbers.end()) {
        std::cout << "找到: " << *it << std::endl;
    }

    // 使用范围适配器
    using namespace boost::adaptors;
    for (int x : numbers | filtered([](int n) { return n % 2 == 0; })) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 范围概念

```cpp
#include <boost/range/algorithm.hpp>
#include <iostream>
#include <vector>
#include <list>
#include <array>

int main() {
    // vector 是范围
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // list 是范围
    std::list<int> lst = {6, 7, 8, 9, 10};

    // 数组是范围
    int arr[] = {11, 12, 13, 14, 15};

    // 使用范围算法
    std::cout << "vec 最大值: " << *boost::max_element(vec) << std::endl;
    std::cout << "lst 最小值: " << *boost::min_element(lst) << std::endl;
    std::cout << "arr 大小: " << boost::size(arr) << std::endl;

    return 0;
}
```

---

## filtered 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 过滤偶数
    std::cout << "偶数: ";
    for (int x : numbers | filtered([](int n) { return n % 2 == 0; })) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 过滤大于5的数
    std::cout << "大于5: ";
    for (int x : numbers | filtered([](int n) { return n > 5; })) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## transformed 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 每个元素乘以2
    std::cout << "乘以2: ";
    for (int x : numbers | transformed([](int n) { return n * 2; })) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 每个元素平方
    std::cout << "平方: ";
    for (int x : numbers | transformed([](int n) { return n * n; })) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## reversed 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 反向遍历
    std::cout << "反向: ";
    for (int x : numbers | reversed) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 字符串反向
    std::string text = "Hello";
    std::cout << "反向文本: ";
    for (char c : text | reversed) {
        std::cout << c;
    }
    std::cout << std::endl;

    return 0;
}
```

---

## sliced 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

    // 切片 [2, 7)
    std::cout << "切片 [2, 7): ";
    for (int x : numbers | sliced(2, 7)) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 跳过前3个
    std::cout << "跳过前3个: ";
    for (int x : numbers | sliced(3, numbers.size())) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 组合适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 先过滤偶数，再乘以3，最后反向
    std::cout << "组合操作: ";
    for (int x : numbers
                | filtered([](int n) { return n % 2 == 0; })
                | transformed([](int n) { return n * 3; })
                | reversed) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## uniqued 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <boost/range/algorithm.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::adaptors;

    std::vector<int> numbers = {1, 1, 2, 2, 2, 3, 3, 4, 5, 5};

    // 去除连续重复元素（需要先排序）
    std::cout << "去重后: ";
    for (int x : numbers | uniqued) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## map_keys 和 map_values

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <map>
#include <string>

int main() {
    using namespace boost::adaptors;

    std::map<std::string, int> ages = {
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35}
    };

    // 只遍历键
    std::cout << "键: ";
    for (const auto& key : ages | map_keys) {
        std::cout << key << " ";
    }
    std::cout << std::endl;

    // 只遍历值
    std::cout << "值: ";
    for (int value : ages | map_values) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## indirected 适配器

```cpp
#include <boost/range/adaptors.hpp>
#include <iostream>
#include <vector>
#include <memory>

int main() {
    using namespace boost::adaptors;

    std::vector<std::shared_ptr<int>> ptrs;
    ptrs.push_back(std::make_shared<int>(10));
    ptrs.push_back(std::make_shared<int>(20));
    ptrs.push_back(std::make_shared<int>(30));

    // 自动解引用指针
    std::cout << "指针指向的值: ";
    for (int value : ptrs | indirected) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

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
    std::vector<int> numbers = {3, 1, 4, 1, 5, 9, 2, 6};

    // 排序
    boost::sort(numbers);
    std::cout << "排序后: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 查找
    auto it = boost::find(numbers, 5);
    if (it != numbers.end()) {
        std::cout << "找到5在位置: " << (it - numbers.begin()) << std::endl;
    }

    // 计数
    int count = boost::count(numbers, 1);
    std::cout << "1出现次数: " << count << std::endl;

    // 累加
    int sum = boost::accumulate(numbers, 0);
    std::cout << "总和: " << sum << std::endl;

    return 0;
}
```

---

## 范围复制

```cpp
#include <boost/range/algorithm.hpp>
#include <iostream>
#include <vector>
#include <list>

int main() {
    std::vector<int> source = {1, 2, 3, 4, 5};
    std::list<int> dest;

    // 复制到 list
    boost::copy(source, std::back_inserter(dest));

    std::cout << "复制后的 list: ";
    for (int x : dest) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 条件复制
    std::vector<int> even_numbers;
    boost::copy_if(source, std::back_inserter(even_numbers),
                   [](int n) { return n % 2 == 0; });

    std::cout << "偶数: ";
    for (int x : even_numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Range 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/range/doc/html/index.html)
- [范围适配器参考](https://www.boost.org/doc/libs/1_90_0/libs/range/doc/html/range/reference/adaptors.html)

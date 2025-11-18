# Boost.Iterator - 迭代器库

## 概述

Boost.Iterator 提供迭代器适配器和工具，简化自定义迭代器的创建。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/iterator/counting_iterator.hpp>
#include <iostream>
#include <algorithm>

int main() {
    using boost::counting_iterator;

    // 创建计数迭代器范围 [0, 10)
    auto begin = counting_iterator<int>(0);
    auto end = counting_iterator<int>(10);

    std::cout << "数字: ";
    for (auto it = begin; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## counting_iterator

```cpp
#include <boost/iterator/counting_iterator.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using boost::counting_iterator;

    // 生成 0 到 9 的序列
    std::vector<int> numbers;
    std::copy(counting_iterator<int>(0),
             counting_iterator<int>(10),
             std::back_inserter(numbers));

    std::cout << "生成的数字: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## filter_iterator

```cpp
#include <boost/iterator/filter_iterator.hpp>
#include <iostream>
#include <vector>

// 偶数谓词
struct is_even {
    bool operator()(int x) const {
        return x % 2 == 0;
    }
};

int main() {
    using boost::filter_iterator;

    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 创建过滤迭代器
    auto begin = filter_iterator<is_even, std::vector<int>::iterator>(
        is_even(), numbers.begin(), numbers.end()
    );
    auto end = filter_iterator<is_even, std::vector<int>::iterator>(
        is_even(), numbers.end(), numbers.end()
    );

    std::cout << "偶数: ";
    for (auto it = begin; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## transform_iterator

```cpp
#include <boost/iterator/transform_iterator.hpp>
#include <iostream>
#include <vector>

// 平方函数对象
struct square {
    int operator()(int x) const {
        return x * x;
    }
};

int main() {
    using boost::transform_iterator;

    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 创建转换迭代器
    auto begin = boost::make_transform_iterator(numbers.begin(), square());
    auto end = boost::make_transform_iterator(numbers.end(), square());

    std::cout << "平方: ";
    for (auto it = begin; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## indirect_iterator

```cpp
#include <boost/iterator/indirect_iterator.hpp>
#include <iostream>
#include <vector>
#include <memory>

int main() {
    using boost::indirect_iterator;

    // 指针向量
    std::vector<std::shared_ptr<int>> ptrs;
    ptrs.push_back(std::make_shared<int>(10));
    ptrs.push_back(std::make_shared<int>(20));
    ptrs.push_back(std::make_shared<int>(30));

    // 创建间接迭代器（自动解引用）
    auto begin = boost::make_indirect_iterator(ptrs.begin());
    auto end = boost::make_indirect_iterator(ptrs.end());

    std::cout << "值: ";
    for (auto it = begin; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## zip_iterator

```cpp
#include <boost/iterator/zip_iterator.hpp>
#include <boost/tuple/tuple.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    using boost::zip_iterator;
    using boost::make_zip_iterator;
    using boost::make_tuple;

    std::vector<std::string> names = {"Alice", "Bob", "Charlie"};
    std::vector<int> ages = {30, 25, 35};

    // 创建 zip 迭代器
    auto begin = make_zip_iterator(make_tuple(names.begin(), ages.begin()));
    auto end = make_zip_iterator(make_tuple(names.end(), ages.end()));

    std::cout << "姓名和年龄:\n";
    for (auto it = begin; it != end; ++it) {
        std::cout << "  " << it->get<0>() << ": " << it->get<1>() << std::endl;
    }

    return 0;
}
```

---

## iterator_facade

```cpp
#include <boost/iterator/iterator_facade.hpp>
#include <iostream>

// 自定义范围迭代器
class range_iterator : public boost::iterator_facade<
    range_iterator,
    int,
    boost::forward_traversal_tag,
    int
> {
public:
    range_iterator() : current_(0) {}
    explicit range_iterator(int start) : current_(start) {}

private:
    friend class boost::iterator_core_access;

    void increment() { ++current_; }

    bool equal(const range_iterator& other) const {
        return current_ == other.current_;
    }

    int dereference() const {
        return current_;
    }

    int current_;
};

int main() {
    range_iterator begin(0);
    range_iterator end(10);

    std::cout << "范围: ";
    for (auto it = begin; it != end; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Iterator 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/iterator/doc/index.html)
- [迭代器适配器参考](https://www.boost.org/doc/libs/1_90_0/libs/iterator/doc/html/index.html)

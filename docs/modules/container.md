# Boost.Container - 高级容器库

## 概述

Boost.Container 提供了 STL 兼容的高性能容器实现。

**类型**: 仅头文件库（大部分）

---

## 快速开始

```cpp
#include <boost/container/vector.hpp>
#include <boost/container/stable_vector.hpp>
#include <boost/container/flat_map.hpp>
#include <iostream>

int main() {
    // 1. vector - 类似 std::vector
    boost::container::vector<int> vec = {1, 2, 3, 4, 5};

    // 2. stable_vector - 稳定的 vector（迭代器不失效）
    boost::container::stable_vector<int> svec = {10, 20, 30};

    // 3. flat_map - 基于排序向量的 map
    boost::container::flat_map<std::string, int> fmap;
    fmap["one"] = 1;
    fmap["two"] = 2;

    for (const auto& [key, value] : fmap) {
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

---

## flat_map 和 flat_set

```cpp
#include <boost/container/flat_map.hpp>
#include <boost/container/flat_set.hpp>
#include <iostream>
#include <string>

int main() {
    // flat_map: 更好的缓存性能
    boost::container::flat_map<int, std::string> scores;

    scores[95] = "Alice";
    scores[87] = "Bob";
    scores[92] = "Charlie";

    std::cout << "Scores (sorted):\n";
    for (const auto& [score, name] : scores) {
        std::cout << score << ": " << name << std::endl;
    }

    // flat_set
    boost::container::flat_set<int> unique_numbers;
    unique_numbers.insert({5, 2, 8, 2, 1, 5});

    std::cout << "\nUnique numbers: ";
    for (int n : unique_numbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## small_vector

```cpp
#include <boost/container/small_vector.hpp>
#include <iostream>

int main() {
    // 小对象优化：前 10 个元素在栈上
    boost::container::small_vector<int, 10> svec;

    for (int i = 0; i < 15; ++i) {
        svec.push_back(i);
    }

    std::cout << "Size: " << svec.size() << std::endl;
    std::cout << "Capacity: " << svec.capacity() << std::endl;

    return 0;
}
```

---

## static_vector

```cpp
#include <boost/container/static_vector.hpp>
#include <iostream>

int main() {
    // 固定容量，栈上分配
    boost::container::static_vector<int, 5> vec;

    vec.push_back(1);
    vec.push_back(2);
    vec.push_back(3);

    std::cout << "Static vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 超过容量会抛出异常
    try {
        for (int i = 0; i < 10; ++i) {
            vec.push_back(i);
        }
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## stable_vector

```cpp
#include <boost/container/stable_vector.hpp>
#include <iostream>

int main() {
    boost::container::stable_vector<int> vec = {1, 2, 3, 4, 5};

    // 保存迭代器
    auto it = vec.begin() + 2;
    std::cout << "Original value: " << *it << std::endl;

    // 插入元素
    vec.insert(vec.begin(), 0);

    // 迭代器仍然有效！
    std::cout << "After insert: " << *it << std::endl;

    return 0;
}
```

---

## devector（双端优化）

```cpp
#include <boost/container/devector.hpp>
#include <iostream>

int main() {
    // devector: 两端都可以高效插入
    boost::container::devector<int> dv;

    dv.push_back(3);
    dv.push_back(4);
    dv.push_front(2);
    dv.push_front(1);

    std::cout << "Devector: ";
    for (int x : dv) {
        std::cout << x << " ";  // 输出: 1 2 3 4
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 性能对比

```cpp
#include <boost/container/flat_map.hpp>
#include <boost/container/vector.hpp>
#include <map>
#include <vector>
#include <chrono>
#include <iostream>

template<typename Container>
void benchmark_insert(const std::string& name, int count) {
    Container container;

    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < count; ++i) {
        container.insert({i, i});
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << name << ": " << duration.count() << " ms" << std::endl;
}

int main() {
    const int COUNT = 100000;

    benchmark_insert<std::map<int, int>>("std::map", COUNT);
    benchmark_insert<boost::container::flat_map<int, int>>("flat_map", COUNT);

    return 0;
}
```

---

## 最佳实践

1. **flat_map**: 查找密集场景
2. **small_vector**: 小容器优化
3. **stable_vector**: 需要稳定迭代器
4. **static_vector**: 嵌入式/实时系统
5. **性能**: 根据访问模式选择

---

## 参考资源

- [Boost.Container 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/container.html)

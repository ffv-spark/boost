# Boost.Bimap - 双向映射库

## 概述

Boost.Bimap 提供双向映射容器，可以从两个方向查找键值对。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost;
    
    // 创建双向映射：ID <-> Name
    bimap<int, std::string> bm;
    
    bm.insert({1, "Alice"});
    bm.insert({2, "Bob"});
    bm.insert({3, "Charlie"});
    
    // 从 ID 查找 Name
    std::cout << "ID 1: " << bm.left.at(1) << std::endl;
    
    // 从 Name 查找 ID
    std::cout << "Alice 的 ID: " << bm.right.at("Alice") << std::endl;
    
    return 0;
}
```

---

## 遍历映射

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost;
    
    bimap<int, std::string> bm;
    bm.insert({1, "Apple"});
    bm.insert({2, "Banana"});
    bm.insert({3, "Cherry"});
    
    std::cout << "从左侧遍历:\\n";
    for (const auto& pair : bm.left) {
        std::cout << "  " << pair.first << " -> " << pair.second << std::endl;
    }
    
    std::cout << "\\n从右侧遍历:\\n";
    for (const auto& pair : bm.right) {
        std::cout << "  " << pair.first << " -> " << pair.second << std::endl;
    }
    
    return 0;
}
```

---

## 多重索引

```cpp
#include <boost/bimap.hpp>
#include <boost/bimap/multiset_of.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::bimaps;
    
    // 允许重复值
    bimap<int, multiset_of<std::string>> bm;
    
    bm.insert({1, "Apple"});
    bm.insert({2, "Banana"});
    bm.insert({3, "Apple"});  // 重复的 "Apple"
    
    std::cout << "ID 到水果:\\n";
    for (const auto& pair : bm.left) {
        std::cout << "  " << pair.first << " -> " << pair.second << std::endl;
    }
    
    // 查找所有 "Apple"
    std::cout << "\\n所有 Apple:\\n";
    auto range = bm.right.equal_range("Apple");
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  ID: " << it->second << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Bimap 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/bimap/doc/html/index.html)

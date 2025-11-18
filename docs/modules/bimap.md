# Boost.Bimap - 双向映射库

## 概述

Boost.Bimap 提供双向映射容器，允许从键查值，也可以从值查键。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    // 创建双向映射：ID <-> 名字
    boost::bimap<int, std::string> bm;

    bm.insert({1, "Alice"});
    bm.insert({2, "Bob"});
    bm.insert({3, "Charlie"});

    // 从左边（ID）查找
    auto it_left = bm.left.find(2);
    if (it_left != bm.left.end()) {
        std::cout << "ID 2 对应的名字: " << it_left->second << std::endl;
    }

    // 从右边（名字）查找
    auto it_right = bm.right.find("Alice");
    if (it_right != bm.right.end()) {
        std::cout << "名字 Alice 对应的 ID: " << it_right->second << std::endl;
    }

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    boost::bimap<int, std::string> bm;

    // 插入元素
    bm.insert({1, "一"});
    bm.insert({2, "二"});
    bm.insert({3, "三"});

    std::cout << "Bimap 大小: " << bm.size() << std::endl;

    // 遍历左视图（按 ID）
    std::cout << "\n按 ID 遍历:\n";
    for (const auto& item : bm.left) {
        std::cout << item.first << " -> " << item.second << std::endl;
    }

    // 遍历右视图（按名字）
    std::cout << "\n按名字遍历:\n";
    for (const auto& item : bm.right) {
        std::cout << item.first << " -> " << item.second << std::endl;
    }

    // 删除元素
    bm.left.erase(2);
    std::cout << "\n删除后大小: " << bm.size() << std::endl;

    return 0;
}
```

---

## 不同的集合类型

```cpp
#include <boost/bimap.hpp>
#include <boost/bimap/set_of.hpp>
#include <boost/bimap/multiset_of.hpp>
#include <boost/bimap/unordered_set_of.hpp>
#include <iostream>
#include <string>

int main() {
    // 使用有序集合（默认）
    typedef boost::bimap<
        boost::bimaps::set_of<int>,
        boost::bimaps::set_of<std::string>
    > ordered_bimap;

    ordered_bimap obm;
    obm.insert({1, "A"});
    obm.insert({2, "B"});

    // 使用无序集合（哈希表）
    typedef boost::bimap<
        boost::bimaps::unordered_set_of<int>,
        boost::bimaps::unordered_set_of<std::string>
    > unordered_bimap;

    unordered_bimap ubm;
    ubm.insert({1, "A"});
    ubm.insert({2, "B"});

    // 使用多重集合（允许重复）
    typedef boost::bimap<
        boost::bimaps::multiset_of<int>,
        boost::bimaps::set_of<std::string>
    > multi_bimap;

    multi_bimap mbm;
    mbm.insert({1, "A"});
    mbm.insert({1, "B"});  // 左侧可以重复

    std::cout << "多重 bimap 大小: " << mbm.size() << std::endl;

    return 0;
}
```

---

## 标记视图

```cpp
#include <boost/bimap.hpp>
#include <boost/bimap/set_of.hpp>
#include <iostream>
#include <string>

// 定义标签
struct id {};
struct name {};

int main() {
    // 使用标签定义 bimap
    typedef boost::bimap<
        boost::bimaps::set_of<boost::bimaps::tagged<int, id>>,
        boost::bimaps::set_of<boost::bimaps::tagged<std::string, name>>
    > tagged_bimap;

    tagged_bimap bm;
    bm.insert({1, "Alice"});
    bm.insert({2, "Bob"});
    bm.insert({3, "Charlie"});

    // 使用标签访问
    auto it_id = bm.by<id>().find(2);
    if (it_id != bm.by<id>().end()) {
        std::cout << "ID 2: " << it_id->get<name>() << std::endl;
    }

    auto it_name = bm.by<name>().find("Alice");
    if (it_name != bm.by<name>().end()) {
        std::cout << "Alice 的 ID: " << it_name->get<id>() << std::endl;
    }

    return 0;
}
```

---

## 字典应用

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

class Dictionary {
public:
    void add(const std::string& english, const std::string& chinese) {
        dict_.insert({english, chinese});
    }

    std::string to_chinese(const std::string& english) const {
        auto it = dict_.left.find(english);
        if (it != dict_.left.end()) {
            return it->second;
        }
        return "未找到";
    }

    std::string to_english(const std::string& chinese) const {
        auto it = dict_.right.find(chinese);
        if (it != dict_.right.end()) {
            return it->second;
        }
        return "未找到";
    }

    void print_all() const {
        std::cout << "英汉词典:\n";
        for (const auto& item : dict_.left) {
            std::cout << item.first << " = " << item.second << std::endl;
        }
    }

private:
    boost::bimap<std::string, std::string> dict_;
};

int main() {
    Dictionary dict;

    dict.add("hello", "你好");
    dict.add("world", "世界");
    dict.add("computer", "计算机");

    std::cout << "hello 的中文: " << dict.to_chinese("hello") << std::endl;
    std::cout << "世界 的英文: " << dict.to_english("世界") << std::endl;

    std::cout << "\n";
    dict.print_all();

    return 0;
}
```

---

## 关系数据

```cpp
#include <boost/bimap.hpp>
#include <boost/bimap/multiset_of.hpp>
#include <iostream>
#include <string>

int main() {
    // 学生 -> 课程 的多对多关系
    typedef boost::bimap<
        boost::bimaps::multiset_of<std::string>,  // 学生
        boost::bimaps::multiset_of<std::string>   // 课程
    > StudentCourses;

    StudentCourses sc;

    // 添加选课关系
    sc.insert({"张三", "数学"});
    sc.insert({"张三", "物理"});
    sc.insert({"张三", "化学"});
    sc.insert({"李四", "数学"});
    sc.insert({"李四", "英语"});
    sc.insert({"王五", "物理"});

    // 查找某学生的所有课程
    std::cout << "张三的课程:\n";
    auto range = sc.left.equal_range("张三");
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->second << std::endl;
    }

    // 查找某课程的所有学生
    std::cout << "\n数学课的学生:\n";
    auto range2 = sc.right.equal_range("数学");
    for (auto it = range2.first; it != range2.second; ++it) {
        std::cout << "  " << it->second << std::endl;
    }

    return 0;
}
```

---

## 投影

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    boost::bimap<int, std::string> bm;

    bm.insert({1, "One"});
    bm.insert({2, "Two"});
    bm.insert({3, "Three"});

    // 从左视图迭代器投影到关系视图
    auto it_left = bm.left.find(2);
    if (it_left != bm.left.end()) {
        // 投影到关系视图
        auto it_relation = bm.project_left(it_left);

        std::cout << "关系: " << it_relation->left << " <-> "
                  << it_relation->right << std::endl;
    }

    // 从右视图迭代器投影
    auto it_right = bm.right.find("Three");
    if (it_right != bm.right.end()) {
        auto it_relation = bm.project_right(it_right);

        std::cout << "关系: " << it_relation->left << " <-> "
                  << it_relation->right << std::endl;
    }

    return 0;
}
```

---

## 修改元素

```cpp
#include <boost/bimap.hpp>
#include <iostream>
#include <string>

int main() {
    boost::bimap<int, std::string> bm;

    bm.insert({1, "One"});
    bm.insert({2, "Two"});
    bm.insert({3, "Three"});

    // 替换右侧值
    auto it = bm.left.find(2);
    if (it != bm.left.end()) {
        bm.left.replace_data(it, "二");
    }

    // 替换左侧键
    auto it2 = bm.right.find("Three");
    if (it2 != bm.right.end()) {
        bm.right.replace_key(it2, 33);
    }

    // 输出修改后的结果
    std::cout << "修改后的 bimap:\n";
    for (const auto& item : bm.left) {
        std::cout << item.first << " -> " << item.second << std::endl;
    }

    return 0;
}
```

---

## 列表类型

```cpp
#include <boost/bimap.hpp>
#include <boost/bimap/list_of.hpp>
#include <iostream>

int main() {
    // 使用列表类型（保持插入顺序）
    typedef boost::bimap<
        boost::bimaps::set_of<int>,
        boost::bimaps::list_of<std::string>
    > list_bimap;

    list_bimap bm;

    bm.insert({3, "Three"});
    bm.insert({1, "One"});
    bm.insert({2, "Two"});

    std::cout << "左视图（有序）:\n";
    for (const auto& item : bm.left) {
        std::cout << item.first << " -> " << item.second << std::endl;
    }

    std::cout << "\n右视图（插入顺序）:\n";
    for (const auto& item : bm.right) {
        std::cout << item.first << " <- " << item.second << std::endl;
    }

    return 0;
}
```

---

## 性能对比

```cpp
#include <boost/bimap.hpp>
#include <map>
#include <iostream>
#include <chrono>

int main() {
    const int size = 100000;

    // 使用 bimap
    auto start1 = std::chrono::high_resolution_clock::now();

    boost::bimap<int, int> bm;
    for (int i = 0; i < size; ++i) {
        bm.insert({i, i * 2});
    }

    // 双向查找
    int sum1 = 0;
    for (int i = 0; i < 1000; ++i) {
        auto it = bm.left.find(i);
        if (it != bm.left.end()) sum1 += it->second;

        auto it2 = bm.right.find(i * 2);
        if (it2 != bm.right.end()) sum1 += it2->second;
    }

    auto end1 = std::chrono::high_resolution_clock::now();
    auto duration1 = std::chrono::duration_cast<std::chrono::milliseconds>(end1 - start1);

    // 使用两个 map
    auto start2 = std::chrono::high_resolution_clock::now();

    std::map<int, int> m1, m2;
    for (int i = 0; i < size; ++i) {
        m1[i] = i * 2;
        m2[i * 2] = i;
    }

    int sum2 = 0;
    for (int i = 0; i < 1000; ++i) {
        auto it = m1.find(i);
        if (it != m1.end()) sum2 += it->second;

        auto it2 = m2.find(i * 2);
        if (it2 != m2.end()) sum2 += it2->second;
    }

    auto end2 = std::chrono::high_resolution_clock::now();
    auto duration2 = std::chrono::duration_cast<std::chrono::milliseconds>(end2 - start2);

    std::cout << "Bimap 耗时: " << duration1.count() << " ms" << std::endl;
    std::cout << "两个 Map 耗时: " << duration2.count() << " ms" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Bimap 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/bimap/doc/html/index.html)
- [Bimap 教程](https://www.boost.org/doc/libs/1_90_0/libs/bimap/doc/html/boost_bimap/the_tutorial.html)

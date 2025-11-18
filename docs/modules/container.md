# Boost.Container - 容器库

## 概述

Boost.Container 提供 STL 兼容的容器实现，包括一些 C++11/14/17 标准容器的扩展版本。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/container/vector.hpp>
#include <iostream>

int main() {
    boost::container::vector<int> vec = {1, 2, 3, 4, 5};
    
    vec.push_back(6);
    
    std::cout << "容器大小: " << vec.size() << std::endl;
    std::cout << "元素: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_container`

---

## flat_map

```cpp
#include <boost/container/flat_map.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::container::flat_map;
    
    // flat_map 使用连续内存存储，查找更快
    flat_map<std::string, int> scores;
    
    scores["Alice"] = 95;
    scores["Bob"] = 87;
    scores["Charlie"] = 92;
    scores["David"] = 88;
    
    std::cout << "成绩表:\\n";
    for (const auto& pair : scores) {
        std::cout << "  " << pair.first << ": " << pair.second << std::endl;
    }
    
    // 查找
    auto it = scores.find("Bob");
    if (it != scores.end()) {
        std::cout << "\\nBob 的成绩: " << it->second << std::endl;
    }
    
    return 0;
}
```

---

## flat_set

```cpp
#include <boost/container/flat_set.hpp>
#include <iostream>

int main() {
    using boost::container::flat_set;
    
    flat_set<int> numbers;
    
    numbers.insert(5);
    numbers.insert(2);
    numbers.insert(8);
    numbers.insert(1);
    numbers.insert(9);
    numbers.insert(2);  // 重复，不会插入
    
    std::cout << "有序集合: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    std::cout << "包含5: " << (numbers.count(5) > 0) << std::endl;
    std::cout << "包含7: " << (numbers.count(7) > 0) << std::endl;
    
    return 0;
}
```

---

## stable_vector

```cpp
#include <boost/container/stable_vector.hpp>
#include <iostream>

int main() {
    using boost::container::stable_vector;
    
    // stable_vector 保证元素地址不变
    stable_vector<int> vec = {10, 20, 30, 40, 50};
    
    int* ptr = &vec[2];  // 指向30
    std::cout << "原始值: " << *ptr << std::endl;
    
    // 插入元素后，指针仍然有效
    vec.push_back(60);
    vec.insert(vec.begin(), 5);
    
    std::cout << "插入后: " << *ptr << std::endl;  // 仍然是30
    
    std::cout << "容器内容: ";
    for (int x : vec) {
        std::cout << x << " ";
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
    using boost::container::small_vector;
    
    // 小容量时使用栈存储，避免堆分配
    small_vector<int, 10> vec;
    
    std::cout << "容量: " << vec.capacity() << std::endl;
    
    for (int i = 0; i < 5; ++i) {
        vec.push_back(i * 10);
    }
    
    std::cout << "元素: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    std::cout << "大小: " << vec.size() << std::endl;
    std::cout << "容量: " << vec.capacity() << std::endl;
    
    return 0;
}
```

---

## static_vector

```cpp
#include <boost/container/static_vector.hpp>
#include <iostream>

int main() {
    using boost::container::static_vector;
    
    // 固定容量的向量，不使用堆内存
    static_vector<int, 5> vec;
    
    vec.push_back(1);
    vec.push_back(2);
    vec.push_back(3);
    
    std::cout << "大小: " << vec.size() << std::endl;
    std::cout << "容量: " << vec.capacity() << std::endl;
    
    std::cout << "元素: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // vec.push_back(6);  // 超过容量会抛出异常
    
    return 0;
}
```

---

## deque

```cpp
#include <boost/container/deque.hpp>
#include <iostream>

int main() {
    using boost::container::deque;
    
    deque<int> dq;
    
    // 两端插入
    dq.push_back(3);
    dq.push_back(4);
    dq.push_front(2);
    dq.push_front(1);
    
    std::cout << "双端队列: ";
    for (int x : dq) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 访问元素
    std::cout << "第一个: " << dq.front() << std::endl;
    std::cout << "最后一个: " << dq.back() << std::endl;
    
    return 0;
}
```

---

## list

```cpp
#include <boost/container/list.hpp>
#include <iostream>

int main() {
    using boost::container::list;
    
    list<int> lst = {1, 2, 3, 4, 5};
    
    // 中间插入
    auto it = lst.begin();
    ++it;
    ++it;
    lst.insert(it, 99);  // 在第3个位置插入
    
    std::cout << "列表: ";
    for (int x : lst) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 删除元素
    lst.remove(99);
    
    std::cout << "删除后: ";
    for (int x : lst) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## map

```cpp
#include <boost/container/map.hpp>
#include <iostream>
#include <string>

int main() {
    using boost::container::map;
    
    map<int, std::string> employees;
    
    employees[1001] = "Alice";
    employees[1002] = "Bob";
    employees[1003] = "Charlie";
    
    std::cout << "员工列表:\\n";
    for (const auto& pair : employees) {
        std::cout << "  ID " << pair.first << ": " << pair.second << std::endl;
    }
    
    // 查找
    if (employees.find(1002) != employees.end()) {
        std::cout << "\\n找到员工1002: " << employees[1002] << std::endl;
    }
    
    return 0;
}
```

---

## set

```cpp
#include <boost/container/set.hpp>
#include <iostream>

int main() {
    using boost::container::set;
    
    set<int> numbers = {5, 2, 8, 1, 9, 3, 7};
    
    std::cout << "有序集合: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 范围查找
    auto lower = numbers.lower_bound(3);
    auto upper = numbers.upper_bound(7);
    
    std::cout << "范围 [3, 7]: ";
    for (auto it = lower; it != upper; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## string

```cpp
#include <boost/container/string.hpp>
#include <iostream>

int main() {
    using boost::container::string;
    
    string str = "Hello";
    str += " ";
    str += "World";
    
    std::cout << "字符串: " << str << std::endl;
    std::cout << "长度: " << str.length() << std::endl;
    
    // 子串
    string sub = str.substr(0, 5);
    std::cout << "子串: " << sub << std::endl;
    
    // 查找
    size_t pos = str.find("World");
    if (pos != string::npos) {
        std::cout << "找到 'World' 在位置: " << pos << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Container 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/container.html)
- [容器性能对比](https://www.boost.org/doc/libs/1_90_0/doc/html/container/containers_of_incomplete_types.html)

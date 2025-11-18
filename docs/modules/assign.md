# Boost.Assign - 赋值库

## 概述

Boost.Assign 提供方便的容器初始化和赋值语法，简化容器的填充操作。

**类型**: 仅头文件库

**注意**: C++11引入了初始化列表，但 Assign 在某些场景仍然有用

---

## 快速开始

```cpp
#include <boost/assign.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::assign;

    // 使用 += 操作符填充 vector
    std::vector<int> vec;
    vec += 1, 2, 3, 4, 5;

    std::cout << "Vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## list_of 函数

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>
#include <list>
#include <set>

int main() {
    using namespace boost::assign;

    // 创建并初始化 vector
    std::vector<int> vec = list_of(1)(2)(3)(4)(5);

    std::cout << "Vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 创建并初始化 list
    std::list<std::string> lst = list_of("apple")("banana")("orange");

    std::cout << "List: ";
    for (const auto& s : lst) {
        std::cout << s << " ";
    }
    std::cout << std::endl;

    // 创建并初始化 set
    std::set<int> s = list_of(3)(1)(4)(1)(5);  // 重复的1会被去除

    std::cout << "Set: ";
    for (int x : s) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## map_list_of 函数

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <map>
#include <string>

int main() {
    using namespace boost::assign;

    // 创建并初始化 map
    std::map<std::string, int> ages = map_list_of
        ("Alice", 30)
        ("Bob", 25)
        ("Charlie", 35);

    std::cout << "Ages:\n";
    for (const auto& p : ages) {
        std::cout << "  " << p.first << ": " << p.second << std::endl;
    }

    return 0;
}
```

---

## push_back 和 push_front

```cpp
#include <boost/assign/std/vector.hpp>
#include <boost/assign/std/deque.hpp>
#include <iostream>
#include <vector>
#include <deque>

int main() {
    using namespace boost::assign;

    // vector 的 push_back
    std::vector<int> vec;
    push_back(vec)(1)(2)(3)(4)(5);

    std::cout << "Vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // deque 的 push_front 和 push_back
    std::deque<int> deq;
    push_front(deq)(5)(4)(3);
    push_back(deq)(1)(2);

    std::cout << "Deque: ";
    for (int x : deq) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## insert 操作

```cpp
#include <boost/assign/std/set.hpp>
#include <boost/assign/std/map.hpp>
#include <iostream>
#include <set>
#include <map>

int main() {
    using namespace boost::assign;

    // set 的 insert
    std::set<int> s;
    insert(s)(1)(2)(3)(4)(5);

    std::cout << "Set: ";
    for (int x : s) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // map 的 insert
    std::map<std::string, int> m;
    insert(m)("one", 1)("two", 2)("three", 3);

    std::cout << "Map:\n";
    for (const auto& p : m) {
        std::cout << "  " << p.first << ": " << p.second << std::endl;
    }

    return 0;
}
```

---

## 范围赋值

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::assign;

    std::vector<int> source = list_of(1)(2)(3)(4)(5);
    std::vector<int> dest;

    // 使用 assign
    dest.assign(source.begin(), source.end());

    std::cout << "Dest: ";
    for (int x : dest) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 重复元素

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::assign;

    // 创建包含重复元素的容器
    std::vector<int> vec = list_of(1).repeat(3, 2)(3);
    // 结果: 1, 2, 2, 2, 3

    std::cout << "Vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 另一种方式
    std::vector<int> vec2 = list_of(0).repeat_from_to(1, 5, 1);
    // 结果: 0, 1, 1, 1, 1, 1

    std::cout << "Vector2: ";
    for (int x : vec2) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 范围生成

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::assign;

    // 生成范围
    std::vector<int> vec = list_of(1).range(2, 6);
    // 结果: 1, 2, 3, 4, 5

    std::cout << "Range: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 与算法结合

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::assign;

    std::vector<int> vec1 = list_of(1)(2)(3)(4)(5);
    std::vector<int> vec2 = list_of(5)(4)(3)(2)(1);

    // 排序
    std::sort(vec2.begin(), vec2.end());

    // 合并
    std::vector<int> result;
    std::merge(vec1.begin(), vec1.end(),
              vec2.begin(), vec2.end(),
              std::back_inserter(result));

    std::cout << "Merged: ";
    for (int x : result) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 初始化二维容器

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::assign;

    // 二维 vector
    std::vector<std::vector<int>> matrix = list_of
        (list_of(1)(2)(3))
        (list_of(4)(5)(6))
        (list_of(7)(8)(9));

    std::cout << "Matrix:\n";
    for (const auto& row : matrix) {
        for (int val : row) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 链式调用

```cpp
#include <boost/assign/std/vector.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::assign;

    std::vector<int> vec;

    // 链式添加元素
    push_back(vec)(1)(2)(3)
                 (4)(5)(6)
                 (7)(8)(9);

    std::cout << "Vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 与 C++11 初始化列表对比

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>
#include <map>

int main() {
    using namespace boost::assign;

    // Boost.Assign 方式
    std::vector<int> vec1 = list_of(1)(2)(3)(4)(5);

    // C++11 初始化列表
    std::vector<int> vec2 = {1, 2, 3, 4, 5};

    // Boost.Assign map
    std::map<std::string, int> map1 = map_list_of("a", 1)("b", 2);

    // C++11 map
    std::map<std::string, int> map2 = {{"a", 1}, {"b", 2}};

    std::cout << "两种方式结果相同" << std::endl;

    // Boost.Assign 在动态添加时更方便
    std::vector<int> vec3;
    vec3 += 1, 2, 3;
    vec3 += 4, 5, 6;

    std::cout << "Dynamic vec3: ";
    for (int x : vec3) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 性能考虑

```cpp
#include <boost/assign/list_of.hpp>
#include <iostream>
#include <vector>
#include <chrono>

int main() {
    using namespace boost::assign;

    const int size = 100000;

    // 使用 Boost.Assign
    auto start1 = std::chrono::high_resolution_clock::now();
    std::vector<int> vec1;
    for (int i = 0; i < size; ++i) {
        vec1 += i;
    }
    auto end1 = std::chrono::high_resolution_clock::now();

    // 使用 push_back
    auto start2 = std::chrono::high_resolution_clock::now();
    std::vector<int> vec2;
    for (int i = 0; i < size; ++i) {
        vec2.push_back(i);
    }
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration_cast<std::chrono::milliseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::milliseconds>(end2 - start2);

    std::cout << "Boost.Assign: " << duration1.count() << " ms" << std::endl;
    std::cout << "push_back: " << duration2.count() << " ms" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Assign 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/assign/doc/index.html)
- [C++11 初始化列表](https://en.cppreference.com/w/cpp/utility/initializer_list)

# Boost.Fusion - 异构容器库

## 概述

Boost.Fusion 提供编译期和运行期的异构数据结构，融合了元编程和运行时编程。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/at_c.hpp>
#include <boost/fusion/include/io.hpp>
#include <iostream>

int main() {
    using namespace boost::fusion;
    
    // 创建异构向量
    vector<int, double, std::string> vec(42, 3.14, "hello");
    
    std::cout << "元素0: " << at_c<0>(vec) << std::endl;
    std::cout << "元素1: " << at_c<1>(vec) << std::endl;
    std::cout << "元素2: " << at_c<2>(vec) << std::endl;
    
    std::cout << "整个向量: " << vec << std::endl;
    
    return 0;
}
```

---

## vector 容器

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/at.hpp>
#include <boost/fusion/include/size.hpp>
#include <iostream>

int main() {
    using namespace boost::fusion;
    
    vector<int, double, char> vec(10, 3.14, 'x');
    
    // 大小
    std::cout << "大小: " << result_of::size<decltype(vec)>::value << std::endl;
    
    // 访问元素
    std::cout << "第0个: " << at_c<0>(vec) << std::endl;
    std::cout << "第1个: " << at_c<1>(vec) << std::endl;
    std::cout << "第2个: " << at_c<2>(vec) << std::endl;
    
    // 修改元素
    at_c<0>(vec) = 20;
    std::cout << "修改后第0个: " << at_c<0>(vec) << std::endl;
    
    return 0;
}
```

---

## map 容器

```cpp
#include <boost/fusion/include/map.hpp>
#include <boost/fusion/include/at_key.hpp>
#include <iostream>
#include <string>

struct name_key {};
struct age_key {};
struct height_key {};

int main() {
    using namespace boost::fusion;
    
    map<
        pair<name_key, std::string>,
        pair<age_key, int>,
        pair<height_key, double>
    > person(
        make_pair<name_key>("Alice"),
        make_pair<age_key>(30),
        make_pair<height_key>(165.5)
    );
    
    std::cout << "姓名: " << at_key<name_key>(person) << std::endl;
    std::cout << "年龄: " << at_key<age_key>(person) << std::endl;
    std::cout << "身高: " << at_key<height_key>(person) << std::endl;
    
    return 0;
}
```

---

## 算法：for_each

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/for_each.hpp>
#include <iostream>

struct print {
    template <typename T>
    void operator()(const T& value) const {
        std::cout << value << " ";
    }
};

int main() {
    using namespace boost::fusion;
    
    vector<int, double, char> vec(42, 3.14, 'x');
    
    std::cout << "遍历元素: ";
    for_each(vec, print());
    std::cout << std::endl;
    
    return 0;
}
```

---

## 算法：transform

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/transform.hpp>
#include <boost/fusion/include/as_vector.hpp>
#include <boost/fusion/include/io.hpp>
#include <iostream>

struct doubler {
    template <typename T>
    T operator()(const T& value) const {
        return value + value;
    }
};

int main() {
    using namespace boost::fusion;
    
    vector<int, double> vec(5, 3.14);
    
    auto result = as_vector(transform(vec, doubler()));
    
    std::cout << "原始: " << vec << std::endl;
    std::cout << "加倍: " << result << std::endl;
    
    return 0;
}
```

---

## 算法：filter

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/filter.hpp>
#include <boost/fusion/include/as_vector.hpp>
#include <boost/fusion/include/io.hpp>
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    using namespace boost::fusion;
    
    vector<int, double, char*, float> vec(10, 3.14, nullptr, 2.5f);
    
    // 过滤出非指针类型
    auto result = as_vector(filter<boost::is_pointer<boost::mpl::_>>(vec));
    
    std::cout << "过滤后: " << result << std::endl;
    
    return 0;
}
```

---

## 序列转换

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/list.hpp>
#include <boost/fusion/include/as_list.hpp>
#include <boost/fusion/include/io.hpp>
#include <iostream>

int main() {
    using namespace boost::fusion;
    
    vector<int, double, char> vec(10, 3.14, 'x');
    
    // vector 转 list
    auto lst = as_list(vec);
    
    std::cout << "vector: " << vec << std::endl;
    std::cout << "list: " << lst << std::endl;
    
    return 0;
}
```

---

## 结构体适配

```cpp
#include <boost/fusion/include/adapt_struct.hpp>
#include <boost/fusion/include/for_each.hpp>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
    double height;
};

BOOST_FUSION_ADAPT_STRUCT(
    Person,
    (std::string, name)
    (int, age)
    (double, height)
)

struct print {
    template <typename T>
    void operator()(const T& value) const {
        std::cout << value << " ";
    }
};

int main() {
    using namespace boost::fusion;
    
    Person p{"Alice", 30, 165.5};
    
    std::cout << "Person 字段: ";
    for_each(p, print());
    std::cout << std::endl;
    
    return 0;
}
```

---

## zip 操作

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/zip.hpp>
#include <boost/fusion/include/for_each.hpp>
#include <boost/fusion/include/io.hpp>
#include <iostream>

int main() {
    using namespace boost::fusion;
    
    vector<int, int, int> v1(1, 2, 3);
    vector<double, double, double> v2(1.1, 2.2, 3.3);
    
    auto zipped = zip(v1, v2);
    
    std::cout << "Zipped: " << zipped << std::endl;
    
    return 0;
}
```

---

## accumulate 累积

```cpp
#include <boost/fusion/include/vector.hpp>
#include <boost/fusion/include/accumulate.hpp>
#include <iostream>

struct sum {
    template <typename T, typename U>
    T operator()(const T& acc, const U& value) const {
        return acc + value;
    }
};

int main() {
    using namespace boost::fusion;
    
    vector<int, int, int> vec(10, 20, 30);
    
    int total = accumulate(vec, 0, sum());
    
    std::cout << "总和: " << total << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Fusion 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/fusion/doc/html/index.html)
- [Fusion 教程](https://www.boost.org/doc/libs/1_90_0/libs/fusion/doc/html/fusion/tutorial.html)

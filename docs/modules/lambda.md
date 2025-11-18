# Boost.Lambda - Lambda表达式库

## 概述

Boost.Lambda 提供匿名函数对象（lambda表达式）支持，是 C++11 lambda 的前身。

**类型**: 仅头文件库

**注意**: C++11 引入了原生 lambda 表达式，优先使用标准 lambda

---

## 快速开始

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // 使用 lambda 打印元素
    std::for_each(numbers.begin(), numbers.end(),
        std::cout << _1 << " "
    );
    std::cout << std::endl;
    
    return 0;
}
```

---

## 算术操作

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::vector<int> results;
    
    // 每个元素乘以2
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  _1 * 2);
    
    std::cout << "乘以2: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 每个元素加10
    results.clear();
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  _1 + 10);
    
    std::cout << "加10: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 条件判断

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/if.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // 统计偶数
    int even_count = std::count_if(numbers.begin(), numbers.end(),
        _1 % 2 == 0
    );
    std::cout << "偶数个数: " << even_count << std::endl;
    
    // 统计大于5的数
    int gt5_count = std::count_if(numbers.begin(), numbers.end(),
        _1 > 5
    );
    std::cout << "大于5的个数: " << gt5_count << std::endl;
    
    return 0;
}
```

---

## 逻辑操作

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // 查找第一个能被2和3整除的数
    auto it = std::find_if(numbers.begin(), numbers.end(),
        _1 % 2 == 0 && _1 % 3 == 0
    );
    
    if (it != numbers.end()) {
        std::cout << "找到: " << *it << std::endl;
    }
    
    // 删除小于3或大于7的元素
    std::vector<int> filtered;
    std::copy_if(numbers.begin(), numbers.end(),
                std::back_inserter(filtered),
                !(_1 < 3 || _1 > 7));
    
    std::cout << "过滤后: ";
    for (int x : filtered) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 绑定值

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3, 7};
    
    int threshold = 5;
    
    // 查找大于阈值的元素
    int count = std::count_if(numbers.begin(), numbers.end(),
        _1 > var(threshold)
    );
    std::cout << "大于 " << threshold << " 的个数: " << count << std::endl;
    
    // 修改阈值
    threshold = 3;
    count = std::count_if(numbers.begin(), numbers.end(),
        _1 > var(threshold)
    );
    std::cout << "大于 " << threshold << " 的个数: " << count << std::endl;
    
    return 0;
}
```

---

## 多参数 Lambda

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> v1 = {1, 2, 3, 4, 5};
    std::vector<int> v2 = {5, 4, 3, 2, 1};
    std::vector<int> results;
    
    // 两个向量相加
    std::transform(v1.begin(), v1.end(), v2.begin(),
                  std::back_inserter(results),
                  _1 + _2);
    
    std::cout << "向量相加: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 两个向量相乘
    results.clear();
    std::transform(v1.begin(), v1.end(), v2.begin(),
                  std::back_inserter(results),
                  _1 * _2);
    
    std::cout << "向量相乘: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 排序

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3, 7};
    
    // 升序排序
    std::sort(numbers.begin(), numbers.end(), _1 < _2);
    
    std::cout << "升序: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    // 降序排序
    std::sort(numbers.begin(), numbers.end(), _1 > _2);
    
    std::cout << "降序: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 成员访问

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

struct Person {
    std::string name;
    int age;
};

int main() {
    using namespace boost::lambda;
    
    std::vector<Person> people = {
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35}
    };
    
    // 按年龄排序
    std::sort(people.begin(), people.end(),
        bind(&Person::age, _1) < bind(&Person::age, _2)
    );
    
    std::cout << "按年龄排序:\\n";
    for (const auto& p : people) {
        std::cout << "  " << p.name << ": " << p.age << std::endl;
    }
    
    return 0;
}
```

---

## 函数调用

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>

int main() {
    using namespace boost::lambda;
    
    std::vector<double> numbers = {1.5, 2.3, 3.7, 4.1};
    std::vector<double> results;
    
    // 应用sqrt函数
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  bind(static_cast<double(*)(double)>(std::sqrt), _1));
    
    std::cout << "平方根: ";
    for (double x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 复合表达式

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::lambda;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::vector<int> results;
    
    // 复杂计算: (x * 2 + 10) / 3
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  (_1 * 2 + 10) / 3);
    
    std::cout << "计算结果: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 与 C++11 Lambda 对比

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // Boost.Lambda
    using namespace boost::lambda;
    std::for_each(numbers.begin(), numbers.end(),
        std::cout << _1 * 2 << " "
    );
    std::cout << std::endl;
    
    // C++11 Lambda (推荐)
    std::for_each(numbers.begin(), numbers.end(),
        [](int x) { std::cout << x * 2 << " "; }
    );
    std::cout << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Lambda 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/lambda.html)
- [C++11 Lambda 表达式](https://en.cppreference.com/w/cpp/language/lambda)

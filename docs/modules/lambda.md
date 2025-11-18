# Boost.Lambda - Lambda 表达式库

## 概述

Boost.Lambda 提供在 C++98/03 中使用 lambda 表达式的能力。

**类型**: 仅头文件库

**注意**: C++11 已引入原生 lambda 表达式，推荐使用 C++11 lambda。此库主要用于遗留代码。

---

## 快速开始

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 使用 Boost.Lambda 遍历
    std::cout << "Numbers: ";
    std::for_each(nums.begin(), nums.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    return 0;
}
```

---

## 占位符

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // _1 表示第一个参数
    // 查找大于 5 的元素
    auto it = std::find_if(nums.begin(), nums.end(), _1 > 5);

    if (it != nums.end()) {
        std::cout << "First number > 5: " << *it << std::endl;
    }

    // 统计偶数
    int even_count = std::count_if(nums.begin(), nums.end(), _1 % 2 == 0);
    std::cout << "Even numbers: " << even_count << std::endl;

    return 0;
}
```

---

## 算术运算

```cpp
#include <boost/lambda/lambda.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    std::vector<int> result(nums.size());

    // 每个元素乘以 2
    std::transform(nums.begin(), nums.end(), result.begin(), _1 * 2);

    std::cout << "Doubled: ";
    std::for_each(result.begin(), result.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    // 每个元素加 10
    std::transform(nums.begin(), nums.end(), result.begin(), _1 + 10);

    std::cout << "Plus 10: ";
    std::for_each(result.begin(), result.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    return 0;
}
```

---

## 逻辑运算

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 统计在 3 到 7 之间的数字
    int count = std::count_if(nums.begin(), nums.end(),
                              _1 >= 3 && _1 <= 7);
    std::cout << "Numbers between 3 and 7: " << count << std::endl;

    // 查找能被 2 或 3 整除的数
    auto it = std::find_if(nums.begin(), nums.end(),
                          _1 % 2 == 0 || _1 % 3 == 0);

    if (it != nums.end()) {
        std::cout << "First number divisible by 2 or 3: " << *it << std::endl;
    }

    return 0;
}
```

---

## 变量绑定

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    int multiplier = 3;

    // 绑定外部变量
    std::vector<int> result(nums.size());
    std::transform(nums.begin(), nums.end(), result.begin(),
                  _1 * var(multiplier));

    std::cout << "Multiplied by " << multiplier << ": ";
    std::for_each(result.begin(), result.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    // 修改外部变量
    int sum = 0;
    std::for_each(nums.begin(), nums.end(), var(sum) += _1);
    std::cout << "Sum: " << sum << std::endl;

    return 0;
}
```

---

## 控制结构

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/if.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {-3, -1, 0, 2, 5, -7, 8};

    // 使用 if 表达式
    std::cout << "Processing: ";
    std::for_each(nums.begin(), nums.end(),
                 if_then_else(_1 > 0,
                            std::cout << constant("positive "),
                            std::cout << constant("non-positive ")));
    std::cout << std::endl;

    // 条件计数
    int positive_count = 0;
    std::for_each(nums.begin(), nums.end(),
                 if_then(_1 > 0, ++var(positive_count)));

    std::cout << "Positive numbers: " << positive_count << std::endl;

    return 0;
}
```

---

## 函数绑定

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>

using namespace boost::lambda;

int compute(int x, int y) {
    return x * x + y;
}

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 绑定普通函数
    std::vector<int> result(nums.size());
    std::transform(nums.begin(), nums.end(), result.begin(),
                  bind(compute, _1, 10));

    std::cout << "Results: ";
    std::for_each(result.begin(), result.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    // 绑定数学函数
    std::vector<double> values = {1.0, 4.0, 9.0, 16.0, 25.0};
    std::vector<double> roots(values.size());

    std::transform(values.begin(), values.end(), roots.begin(),
                  bind(std::sqrt, _1));

    std::cout << "Square roots: ";
    std::for_each(roots.begin(), roots.end(), std::cout << _1 << " ");
    std::cout << std::endl;

    return 0;
}
```

---

## 成员函数调用

```cpp
#include <boost/lambda/lambda.hpp>
#include <boost/lambda/bind.hpp>
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

using namespace boost::lambda;

class Person {
public:
    Person(const std::string& name, int age) : name_(name), age_(age) {}

    std::string get_name() const { return name_; }
    int get_age() const { return age_; }

    void print() const {
        std::cout << name_ << " (" << age_ << ")" << std::endl;
    }

private:
    std::string name_;
    int age_;
};

int main() {
    std::vector<Person> people = {
        Person("Alice", 30),
        Person("Bob", 25),
        Person("Charlie", 35)
    };

    // 调用成员函数
    std::cout << "People:\n";
    std::for_each(people.begin(), people.end(),
                 bind(&Person::print, _1));

    // 使用成员函数进行排序
    std::sort(people.begin(), people.end(),
             bind(&Person::get_age, _1) < bind(&Person::get_age, _2));

    std::cout << "\nSorted by age:\n";
    std::for_each(people.begin(), people.end(),
                 bind(&Person::print, _1));

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

using namespace boost::lambda;

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    std::cout << "Boost.Lambda 方式:\n";
    {
        int count = std::count_if(nums.begin(), nums.end(), _1 % 2 == 0);
        std::cout << "Even numbers: " << count << std::endl;
    }

    std::cout << "\nC++11 Lambda 方式:\n";
    {
        int count = std::count_if(nums.begin(), nums.end(),
                                 [](int x) { return x % 2 == 0; });
        std::cout << "Even numbers: " << count << std::endl;
    }

    // Boost.Lambda
    std::cout << "\nBoost.Lambda transform:\n";
    {
        std::vector<int> result(nums.size());
        std::transform(nums.begin(), nums.end(), result.begin(), _1 * 2);

        std::for_each(result.begin(), result.end(), std::cout << _1 << " ");
        std::cout << std::endl;
    }

    // C++11 Lambda
    std::cout << "\nC++11 Lambda transform:\n";
    {
        std::vector<int> result(nums.size());
        std::transform(nums.begin(), nums.end(), result.begin(),
                      [](int x) { return x * 2; });

        for (int x : result) {
            std::cout << x << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

**推荐**: 对于现代 C++ 项目，使用 C++11/14/17 的原生 lambda 表达式，它们更简洁、更易读。

---

## 参考资源

- [Boost.Lambda 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/lambda.html)
- [C++11 Lambda 表达式](https://en.cppreference.com/w/cpp/language/lambda)

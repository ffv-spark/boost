# Boost.Phoenix - 函数式编程库

## 概述

Boost.Phoenix 提供函数式编程工具，支持 lambda 表达式、lazy 函数和函数组合。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // 使用 phoenix lambda
    std::for_each(numbers.begin(), numbers.end(),
        std::cout << arg1 * 2 << " "
    );
    std::cout << std::endl;
    
    return 0;
}
```

---

## Lambda 表达式

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::vector<int> results;
    
    // 每个元素乘以2
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  arg1 * 2);
    
    std::cout << "乘以2: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 条件表达式

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // 统计偶数
    int count = std::count_if(numbers.begin(), numbers.end(),
        arg1 % 2 == 0
    );
    
    std::cout << "偶数个数: " << count << std::endl;
    
    // 统计大于5的数
    count = std::count_if(numbers.begin(), numbers.end(),
        arg1 > 5
    );
    
    std::cout << "大于5的个数: " << count << std::endl;
    
    return 0;
}
```

---

## 逻辑运算

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // 查找能被2和3整除的数
    auto it = std::find_if(numbers.begin(), numbers.end(),
        (arg1 % 2 == 0) && (arg1 % 3 == 0)
    );
    
    if (it != numbers.end()) {
        std::cout << "找到: " << *it << std::endl;
    }
    
    // 过滤：3到7之间的数
    std::vector<int> filtered;
    std::copy_if(numbers.begin(), numbers.end(),
                std::back_inserter(filtered),
                (arg1 >= 3) && (arg1 <= 7));
    
    std::cout << "3到7之间: ";
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
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3, 7};
    
    int threshold = 5;
    
    // 使用外部变量
    int count = std::count_if(numbers.begin(), numbers.end(),
        arg1 > val(threshold)
    );
    
    std::cout << "大于 " << threshold << " 的个数: " << count << std::endl;
    
    // 使用引用
    int sum = 0;
    std::for_each(numbers.begin(), numbers.end(),
        ref(sum) += arg1
    );
    
    std::cout << "总和: " << sum << std::endl;
    
    return 0;
}
```

---

## 函数调用

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<double> numbers = {1.5, 2.3, 3.7, 4.1};
    std::vector<double> results;
    
    // 应用sqrt函数
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  bind(static_cast<double(*)(double)>(std::sqrt), arg1));
    
    std::cout << "平方根: ";
    for (double x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 成员函数调用

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<std::string> words = {"hello", "world", "boost", "phoenix"};
    
    // 打印每个字符串的长度
    std::cout << "字符串长度: ";
    std::for_each(words.begin(), words.end(),
        std::cout << bind(&std::string::length, arg1) << " "
    );
    std::cout << std::endl;
    
    // 按长度排序
    std::sort(words.begin(), words.end(),
        bind(&std::string::length, arg1) < bind(&std::string::length, arg2)
    );
    
    std::cout << "按长度排序: ";
    for (const auto& word : words) {
        std::cout << word << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 复合表达式

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::vector<int> results;
    
    // 复杂计算: (x * 2 + 10) / 3
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  (arg1 * 2 + 10) / 3);
    
    std::cout << "计算结果: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## if_else 表达式

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::vector<int> results;
    
    // 偶数返回原值，奇数返回0
    std::transform(numbers.begin(), numbers.end(),
                  std::back_inserter(results),
                  if_(arg1 % 2 == 0)[arg1].else_[0]);
    
    std::cout << "结果: ";
    for (int x : results) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## Lazy 函数

```cpp
#include <boost/phoenix.hpp>
#include <iostream>

struct calculator {
    int add(int a, int b) const {
        return a + b;
    }
    
    int multiply(int a, int b) const {
        return a * b;
    }
};

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    
    calculator calc;
    
    // 创建 lazy 函数
    auto lazy_add = bind(&calculator::add, &calc, arg1, arg2);
    auto lazy_mul = bind(&calculator::multiply, &calc, arg1, arg2);
    
    std::cout << "5 + 3 = " << lazy_add(5, 3) << std::endl;
    std::cout << "5 * 3 = " << lazy_mul(5, 3) << std::endl;
    
    // 组合
    auto combined = lazy_add(lazy_mul(2, 3), 10);
    std::cout << "(2 * 3) + 10 = " << combined() << std::endl;
    
    return 0;
}
```

---

## 局部变量

```cpp
#include <boost/phoenix.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::phoenix;
    using namespace boost::phoenix::arg_names;
    using namespace boost::phoenix::local_names;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // 使用局部变量计算平方和
    int sum = 0;
    std::for_each(numbers.begin(), numbers.end(),
        let(_a = arg1 * arg1)[
            ref(sum) += _a
        ]
    );
    
    std::cout << "平方和: " << sum << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Phoenix 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/phoenix/doc/html/index.html)
- [Phoenix 教程](https://www.boost.org/doc/libs/1_90_0/libs/phoenix/doc/html/phoenix/starter_kit.html)

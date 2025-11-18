# Boost.Optional - 可选值库

## 概述

Boost.Optional 提供可选值类型，表示一个值可能存在也可能不存在。

**类型**: 仅头文件库

**注意**: C++17 引入了 std::optional，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

int main() {
    // 创建可选值
    boost::optional<int> opt1 = 42;
    boost::optional<int> opt2;  // 空

    // 检查是否有值
    if (opt1) {
        std::cout << "opt1 有值: " << *opt1 << std::endl;
    }

    if (!opt2) {
        std::cout << "opt2 为空" << std::endl;
    }

    return 0;
}
```

---

## 创建可选值

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

int main() {
    // 默认构造（空值）
    boost::optional<int> opt1;

    // 从值构造
    boost::optional<int> opt2(42);
    boost::optional<int> opt3 = 100;

    // 使用 boost::none
    boost::optional<int> opt4 = boost::none;

    // 使用 make_optional
    auto opt5 = boost::make_optional(3.14);

    std::cout << std::boolalpha;
    std::cout << "opt1 有值: " << bool(opt1) << std::endl;
    std::cout << "opt2 有值: " << bool(opt2) << std::endl;

    return 0;
}
```

---

## 访问值

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> opt = 42;

    // 使用 * 解引用
    if (opt) {
        std::cout << "值: " << *opt << std::endl;
    }

    // 使用 get()
    if (opt) {
        std::cout << "get(): " << opt.get() << std::endl;
    }

    // 使用 get_value_or() 提供默认值
    boost::optional<int> empty;
    std::cout << "有默认值: " << empty.get_value_or(0) << std::endl;

    return 0;
}
```

---

## 条件返回

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>
#include <vector>

// 查找第一个偶数
boost::optional<int> find_first_even(const std::vector<int>& vec) {
    for (int x : vec) {
        if (x % 2 == 0) {
            return x;  // 找到
        }
    }
    return boost::none;  // 未找到
}

int main() {
    std::vector<int> numbers1 = {1, 3, 5, 7, 9};
    std::vector<int> numbers2 = {1, 3, 4, 7, 9};

    auto result1 = find_first_even(numbers1);
    auto result2 = find_first_even(numbers2);

    if (result1) {
        std::cout << "找到偶数: " << *result1 << std::endl;
    } else {
        std::cout << "未找到偶数" << std::endl;
    }

    if (result2) {
        std::cout << "找到偶数: " << *result2 << std::endl;
    } else {
        std::cout << "未找到偶数" << std::endl;
    }

    return 0;
}
```

---

## 赋值操作

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> opt;

    std::cout << std::boolalpha;
    std::cout << "初始为空: " << !opt << std::endl;

    // 赋值
    opt = 42;
    std::cout << "赋值后: " << *opt << std::endl;

    // 重置为空
    opt = boost::none;
    std::cout << "重置后为空: " << !opt << std::endl;

    // 再次赋值
    opt = 100;
    std::cout << "再次赋值: " << *opt << std::endl;

    return 0;
}
```

---

## 比较操作

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> opt1 = 42;
    boost::optional<int> opt2 = 42;
    boost::optional<int> opt3 = 100;
    boost::optional<int> opt4;

    std::cout << std::boolalpha;

    // 与其他 optional 比较
    std::cout << "opt1 == opt2: " << (opt1 == opt2) << std::endl;
    std::cout << "opt1 != opt3: " << (opt1 != opt3) << std::endl;
    std::cout << "opt1 < opt3: " << (opt1 < opt3) << std::endl;

    // 与 none 比较
    std::cout << "opt1 == none: " << (opt1 == boost::none) << std::endl;
    std::cout << "opt4 == none: " << (opt4 == boost::none) << std::endl;

    // 与值直接比较
    std::cout << "opt1 == 42: " << (opt1 == 42) << std::endl;

    return 0;
}
```

---

## 指针语义

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
};

int main() {
    boost::optional<Person> opt = Person{"Alice", 30};

    // 使用 -> 访问成员
    if (opt) {
        std::cout << "姓名: " << opt->name << std::endl;
        std::cout << "年龄: " << opt->age << std::endl;
    }

    // 修改成员
    opt->age = 31;
    std::cout << "修改后年龄: " << opt->age << std::endl;

    return 0;
}
```

---

## 惰性初始化

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <vector>

class ExpensiveObject {
public:
    ExpensiveObject() {
        std::cout << "创建昂贵对象" << std::endl;
        data.resize(1000000);
    }

    void use() {
        std::cout << "使用对象" << std::endl;
    }

private:
    std::vector<int> data;
};

class Container {
public:
    void use_expensive_object() {
        if (!expensive) {
            expensive = ExpensiveObject();  // 惰性初始化
        }
        expensive->use();
    }

private:
    boost::optional<ExpensiveObject> expensive;
};

int main() {
    Container c;

    std::cout << "创建容器（不创建昂贵对象）" << std::endl;

    std::cout << "第一次使用..." << std::endl;
    c.use_expensive_object();

    std::cout << "第二次使用..." << std::endl;
    c.use_expensive_object();

    return 0;
}
```

---

## 配置值

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

struct Config {
    std::string host = "localhost";
    int port = 8080;
    boost::optional<std::string> username;  // 可选
    boost::optional<std::string> password;  // 可选
};

void connect(const Config& config) {
    std::cout << "连接到 " << config.host << ":" << config.port << std::endl;

    if (config.username) {
        std::cout << "用户名: " << *config.username << std::endl;
        if (config.password) {
            std::cout << "使用密码认证" << std::endl;
        }
    } else {
        std::cout << "匿名连接" << std::endl;
    }
}

int main() {
    Config config1;
    std::cout << "配置1（匿名）:\n";
    connect(config1);

    Config config2;
    config2.username = "admin";
    config2.password = "secret";
    std::cout << "\n配置2（认证）:\n";
    connect(config2);

    return 0;
}
```

---

## 链式操作

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

// 解析整数
boost::optional<int> parse_int(const std::string& s) {
    try {
        return std::stoi(s);
    } catch (...) {
        return boost::none;
    }
}

// 检查是否为正数
boost::optional<int> check_positive(int x) {
    return x > 0 ? boost::optional<int>(x) : boost::none;
}

int main() {
    std::string input1 = "42";
    std::string input2 = "-10";
    std::string input3 = "abc";

    // 链式检查
    auto result1 = parse_int(input1);
    if (result1) {
        result1 = check_positive(*result1);
    }

    auto result2 = parse_int(input2);
    if (result2) {
        result2 = check_positive(*result2);
    }

    auto result3 = parse_int(input3);

    std::cout << "42 解析并检查: " << (result1 ? "成功" : "失败") << std::endl;
    std::cout << "-10 解析并检查: " << (result2 ? "成功" : "失败") << std::endl;
    std::cout << "abc 解析: " << (result3 ? "成功" : "失败") << std::endl;

    return 0;
}
```

---

## 容器中的可选值

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <vector>
#include <string>

struct Student {
    std::string name;
    boost::optional<double> grade;  // 可能还没有成绩
};

int main() {
    std::vector<Student> students = {
        {"Alice", 85.5},
        {"Bob", boost::none},  // 还没考试
        {"Charlie", 92.0}
    };

    std::cout << "学生成绩:\n";
    for (const auto& student : students) {
        std::cout << "  " << student.name << ": ";
        if (student.grade) {
            std::cout << *student.grade << std::endl;
        } else {
            std::cout << "未考试" << std::endl;
        }
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Optional 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/optional/doc/html/index.html)
- [std::optional 参考](https://en.cppreference.com/w/cpp/utility/optional)

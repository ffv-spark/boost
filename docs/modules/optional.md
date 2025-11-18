# Boost.Optional - 可选值库

## 概述

Boost.Optional 提供了一个类型安全的可选值容器，表示一个值可能存在也可能不存在。

**类型**: 仅头文件库

**注意**: C++17 已引入 `std::optional`

**主要特性**:
- 避免使用指针或特殊值表示"无值"
- 类型安全
- 明确的语义
- 零开销抽象

---

## 快速开始

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

boost::optional<int> divide(int a, int b) {
    if (b == 0) {
        return boost::none;  // 返回空值
    }
    return a / b;
}

int main() {
    auto result = divide(10, 2);

    if (result) {
        std::cout << "结果: " << *result << std::endl;
    } else {
        std::cout << "除法失败" << std::endl;
    }

    auto failed = divide(10, 0);
    std::cout << "值: " << failed.value_or(-1) << std::endl;  // 使用默认值

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -o example
```

---

## 基本用法

### 创建 Optional

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

int main() {
    // 1. 空值
    boost::optional<int> empty;
    boost::optional<int> empty2 = boost::none;
    boost::optional<int> empty3(boost::none);

    // 2. 有值
    boost::optional<int> value1(42);
    boost::optional<int> value2 = 42;
    boost::optional<std::string> str("hello");

    // 3. 使用 make_optional
    auto value3 = boost::make_optional(100);
    auto str2 = boost::make_optional<std::string>("world");

    // 4. 条件创建
    bool condition = true;
    auto conditional = boost::make_optional(condition, 42);

    std::cout << "empty: " << (empty ? "有值" : "无值") << std::endl;
    std::cout << "value1: " << *value1 << std::endl;
    std::cout << "conditional: " << (conditional ? std::to_string(*conditional) : "无值") << std::endl;

    return 0;
}
```

### 检查和访问值

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> opt = 42;

    // 1. 检查是否有值
    if (opt) {
        std::cout << "有值" << std::endl;
    }

    if (opt.is_initialized()) {
        std::cout << "已初始化" << std::endl;
    }

    // 2. 获取值
    int value1 = *opt;  // 解引用
    int value2 = opt.get();  // get() 方法
    int value3 = opt.get_value_or(0);  // 带默认值
    int value4 = opt.value_or(0);  // 简写形式

    std::cout << "值: " << value1 << std::endl;

    // 3. 获取指针
    int* ptr = opt.get_ptr();
    if (ptr) {
        std::cout << "通过指针: " << *ptr << std::endl;
    }

    // 4. 空值处理
    boost::optional<int> empty;
    // int bad = *empty;  // 未定义行为！
    int safe = empty.value_or(100);  // 安全，返回 100

    return 0;
}
```

### 修改值

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> opt;

    // 1. 赋值
    opt = 42;
    std::cout << "赋值后: " << *opt << std::endl;

    // 2. 重置为空
    opt = boost::none;
    std::cout << "重置后: " << (opt ? "有值" : "无值") << std::endl;

    // 3. reset() 方法
    opt = 100;
    opt.reset();
    std::cout << "reset后: " << (opt ? "有值" : "无值") << std::endl;

    // 4. 就地构造
    boost::optional<std::string> str;
    str.emplace("Hello");  // 直接在内部构造
    std::cout << "emplace后: " << *str << std::endl;

    return 0;
}
```

---

## 实用示例

### 查找操作

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <vector>
#include <string>

boost::optional<std::string> find_user(int id) {
    static std::vector<std::pair<int, std::string>> users = {
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"}
    };

    for (const auto& [uid, name] : users) {
        if (uid == id) {
            return name;
        }
    }

    return boost::none;
}

int main() {
    auto user = find_user(2);

    if (user) {
        std::cout << "找到用户: " << *user << std::endl;
    } else {
        std::cout << "用户不存在" << std::endl;
    }

    // 使用默认值
    std::string name = find_user(999).value_or("Unknown");
    std::cout << "用户名: " << name << std::endl;

    return 0;
}
```

### 配置解析

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <map>
#include <string>

class Config {
public:
    void set(const std::string& key, const std::string& value) {
        config_[key] = value;
    }

    boost::optional<std::string> get_string(const std::string& key) const {
        auto it = config_.find(key);
        if (it != config_.end()) {
            return it->second;
        }
        return boost::none;
    }

    boost::optional<int> get_int(const std::string& key) const {
        auto str = get_string(key);
        if (str) {
            try {
                return std::stoi(*str);
            } catch (...) {
                return boost::none;
            }
        }
        return boost::none;
    }

private:
    std::map<std::string, std::string> config_;
};

int main() {
    Config config;
    config.set("host", "localhost");
    config.set("port", "8080");
    config.set("debug", "true");

    std::string host = config.get_string("host").value_or("0.0.0.0");
    int port = config.get_int("port").value_or(80);
    int timeout = config.get_int("timeout").value_or(30);

    std::cout << "主机: " << host << std::endl;
    std::cout << "端口: " << port << std::endl;
    std::cout << "超时: " << timeout << " 秒" << std::endl;

    return 0;
}
```

### 链式调用

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <string>

struct Address {
    std::string city;
    boost::optional<std::string> postal_code;
};

struct Person {
    std::string name;
    boost::optional<Address> address;
};

int main() {
    Person p1{"Alice", Address{"Beijing", "100000"}};
    Person p2{"Bob", boost::none};

    // 安全的链式访问
    auto postal1 = p1.address
        ? p1.address->postal_code
        : boost::none;

    auto postal2 = p2.address
        ? p2.address->postal_code
        : boost::none;

    std::cout << "Alice 邮编: "
              << postal1.value_or("未知") << std::endl;

    std::cout << "Bob 邮编: "
              << postal2.value_or("未知") << std::endl;

    return 0;
}
```

### 惰性计算

```cpp
#include <boost/optional.hpp>
#include <iostream>
#include <cmath>

class Calculator {
public:
    boost::optional<double> cached_sqrt(double value) {
        if (value < 0) {
            return boost::none;
        }

        // 检查缓存
        if (cache_ && cache_input_ == value) {
            std::cout << "使用缓存值" << std::endl;
            return cache_;
        }

        // 计算并缓存
        std::cout << "计算平方根" << std::endl;
        cache_input_ = value;
        cache_ = std::sqrt(value);

        return cache_;
    }

private:
    double cache_input_ = 0;
    boost::optional<double> cache_;
};

int main() {
    Calculator calc;

    auto result1 = calc.cached_sqrt(16);
    std::cout << "√16 = " << *result1 << "\n" << std::endl;

    auto result2 = calc.cached_sqrt(16);  // 使用缓存
    std::cout << "√16 = " << *result2 << "\n" << std::endl;

    auto result3 = calc.cached_sqrt(-1);
    if (!result3) {
        std::cout << "无法计算负数的平方根" << std::endl;
    }

    return 0;
}
```

---

## 高级用法

### Optional 引用

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    int value = 42;

    // Optional 引用
    boost::optional<int&> opt_ref(value);

    std::cout << "原始值: " << value << std::endl;
    std::cout << "Optional 值: " << *opt_ref << std::endl;

    // 修改引用
    *opt_ref = 100;
    std::cout << "修改后原始值: " << value << std::endl;

    return 0;
}
```

### 比较操作

```cpp
#include <boost/optional.hpp>
#include <iostream>

int main() {
    boost::optional<int> a = 10;
    boost::optional<int> b = 20;
    boost::optional<int> c = 10;
    boost::optional<int> empty;

    // Optional 之间的比较
    std::cout << "a == c: " << (a == c) << std::endl;
    std::cout << "a < b: " << (a < b) << std::endl;

    // 与值的比较
    std::cout << "a == 10: " << (a == 10) << std::endl;
    std::cout << "b > 15: " << (b > 15) << std::endl;

    // 与 none 的比较
    std::cout << "empty == none: " << (empty == boost::none) << std::endl;
    std::cout << "a != none: " << (a != boost::none) << std::endl;

    // 空值比较
    boost::optional<int> e1, e2;
    std::cout << "两个空值相等: " << (e1 == e2) << std::endl;

    return 0;
}
```

---

## 与 std::optional 的区别

```cpp
#include <boost/optional.hpp>
#include <optional>  // C++17
#include <iostream>

int main() {
    // Boost.Optional
    boost::optional<int> b_opt = 42;
    int b_val = b_opt.get_value_or(0);

    // std::optional (C++17)
    std::optional<int> s_opt = 42;
    int s_val = s_opt.value_or(0);

    // 主要区别：
    // 1. boost::optional 兼容 C++98
    // 2. std::optional 有 value() 方法（抛出异常）
    // 3. std::optional 有 has_value() 而不是 is_initialized()
    // 4. boost::optional 支持引用类型

    std::cout << "Boost: " << b_val << std::endl;
    std::cout << "Std: " << s_val << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **优先使用 value_or()**: 避免解引用空值
2. **避免存储指针**: Optional 本身就是指针语义
3. **明确语义**: 用 Optional 表达"可能不存在"的概念
4. **异常安全**: 使用 get_value_or() 而不是直接解引用
5. **C++17+**: 优先使用 std::optional

---

## 参考资源

- [Boost.Optional 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/optional/doc/html/index.html)

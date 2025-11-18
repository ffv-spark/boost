# Boost.Any - 任意类型容器

## 概述

Boost.Any 提供了类型安全的任意类型值容器。

**类型**: 仅头文件库

**注意**: C++17 已引入 `std::any`

---

## 快速开始

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    // 创建 any
    boost::any a = 42;
    boost::any b = std::string("Hello");
    boost::any c = 3.14;

    // 获取值
    int i = boost::any_cast<int>(a);
    std::string s = boost::any_cast<std::string>(b);
    double d = boost::any_cast<double>(c);

    std::cout << "int: " << i << std::endl;
    std::cout << "string: " << s << std::endl;
    std::cout << "double: " << d << std::endl;

    // 检查类型
    if (a.type() == typeid(int)) {
        std::cout << "a is int" << std::endl;
    }

    // 判空
    boost::any empty;
    std::cout << "empty: " << empty.empty() << std::endl;

    return 0;
}
```

---

## 类型转换

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <string>

int main() {
    boost::any value = 42;

    // 方法1: any_cast（值）
    try {
        int i = boost::any_cast<int>(value);
        std::cout << "Value: " << i << std::endl;

        // 错误的类型会抛出异常
        std::string s = boost::any_cast<std::string>(value);
    } catch (boost::bad_any_cast& e) {
        std::cerr << "Bad cast: " << e.what() << std::endl;
    }

    // 方法2: any_cast（指针，安全）
    if (int* p = boost::any_cast<int>(&value)) {
        std::cout << "Pointer cast: " << *p << std::endl;
    }

    if (boost::any_cast<std::string>(&value) == nullptr) {
        std::cout << "Not a string" << std::endl;
    }

    return 0;
}
```

---

## 实用示例

### 异构容器

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<boost::any> values;

    values.push_back(42);
    values.push_back(std::string("Hello"));
    values.push_back(3.14);
    values.push_back(true);

    for (const auto& v : values) {
        if (v.type() == typeid(int)) {
            std::cout << "int: " << boost::any_cast<int>(v) << std::endl;
        } else if (v.type() == typeid(std::string)) {
            std::cout << "string: " << boost::any_cast<std::string>(v) << std::endl;
        } else if (v.type() == typeid(double)) {
            std::cout << "double: " << boost::any_cast<double>(v) << std::endl;
        } else if (v.type() == typeid(bool)) {
            std::cout << "bool: " << boost::any_cast<bool>(v) << std::endl;
        }
    }

    return 0;
}
```

### 配置存储

```cpp
#include <boost/any.hpp>
#include <iostream>
#include <map>
#include <string>

class Config {
public:
    template<typename T>
    void set(const std::string& key, const T& value) {
        data_[key] = value;
    }

    template<typename T>
    T get(const std::string& key, const T& default_value) const {
        auto it = data_.find(key);
        if (it != data_.end()) {
            try {
                return boost::any_cast<T>(it->second);
            } catch (boost::bad_any_cast&) {
                return default_value;
            }
        }
        return default_value;
    }

private:
    std::map<std::string, boost::any> data_;
};

int main() {
    Config config;

    config.set("port", 8080);
    config.set("host", std::string("localhost"));
    config.set("debug", true);
    config.set("timeout", 30.5);

    int port = config.get("port", 80);
    std::string host = config.get("host", std::string("0.0.0.0"));
    bool debug = config.get("debug", false);
    double timeout = config.get("timeout", 10.0);

    std::cout << "Port: " << port << std::endl;
    std::cout << "Host: " << host << std::endl;
    std::cout << "Debug: " << debug << std::endl;
    std::cout << "Timeout: " << timeout << std::endl;

    return 0;
}
```

---

## 与 std::any 的对比

```cpp
#include <boost/any.hpp>
#include <any>  // C++17
#include <iostream>

int main() {
    // Boost.Any
    boost::any b_any = 42;
    int b_val = boost::any_cast<int>(b_any);

    // std::any (C++17)
    std::any s_any = 42;
    int s_val = std::any_cast<int>(s_any);

    // 主要相同，API 几乎一致
    std::cout << "Boost: " << b_val << std::endl;
    std::cout << "Std: " << s_val << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **优先使用 variant**: 如果类型集合已知
2. **检查类型**: 使用 type() 或指针转换
3. **异常处理**: 捕获 bad_any_cast
4. **性能考虑**: 涉及动态分配
5. **C++17+**: 优先使用 std::any

---

## 参考资源

- [Boost.Any 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/any.html)

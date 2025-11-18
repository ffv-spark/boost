# Boost.JSON - JSON 解析库

## 概述

Boost.JSON 是一个快速、现代的 JSON 解析和生成库，专为 C++11 及以上设计。

**类型**: 需要编译链接的库

**链接库**: `-lboost_json`

**特性**:
- 快速解析和序列化
- 低内存占用
- 支持自定义内存分配器
- 异常安全

---

## 快速开始

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    // 解析 JSON
    json::value jv = json::parse(R"({"name": "John", "age": 30})");

    std::cout << "姓名: " << jv.at("name").as_string() << std::endl;
    std::cout << "年龄: " << jv.at("age").as_int64() << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_json -o example
```

---

## 创建 JSON 值

### 基本类型

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    // 1. null
    json::value v1 = nullptr;

    // 2. bool
    json::value v2 = true;

    // 3. 数字
    json::value v3 = 42;
    json::value v4 = 3.14;

    // 4. 字符串
    json::value v5 = "Hello, JSON!";

    // 5. 数组
    json::value v6 = {1, 2, 3, 4, 5};

    // 6. 对象
    json::value v7 = {
        {"name", "John"},
        {"age", 30},
        {"city", "New York"}
    };

    std::cout << v7 << std::endl;

    return 0;
}
```

### 构建复杂对象

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    json::object person;

    // 添加基本字段
    person["name"] = "Alice";
    person["age"] = 28;
    person["email"] = "alice@example.com";

    // 添加数组
    json::array hobbies;
    hobbies.push_back("reading");
    hobbies.push_back("hiking");
    hobbies.push_back("coding");
    person["hobbies"] = hobbies;

    // 添加嵌套对象
    json::object address;
    address["city"] = "Beijing";
    address["country"] = "China";
    person["address"] = address;

    // 序列化为字符串
    std::cout << json::serialize(person) << std::endl;

    return 0;
}
```

---

## 解析 JSON

### 从字符串解析

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    std::string json_str = R"({
        "name": "Bob",
        "age": 35,
        "skills": ["C++", "Python", "JavaScript"],
        "active": true
    })";

    // 解析
    json::value jv = json::parse(json_str);

    // 访问字段
    std::cout << "姓名: " << jv.at("name").as_string() << std::endl;
    std::cout << "年龄: " << jv.at("age").as_int64() << std::endl;
    std::cout << "状态: " << (jv.at("active").as_bool() ? "活跃" : "非活跃") << std::endl;

    // 遍历数组
    std::cout << "技能: ";
    for (const auto& skill : jv.at("skills").as_array()) {
        std::cout << skill.as_string() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 从文件解析

```cpp
#include <boost/json.hpp>
#include <iostream>
#include <fstream>
#include <sstream>

namespace json = boost::json;

int main() {
    // 读取文件
    std::ifstream file("config.json");
    std::stringstream buffer;
    buffer << file.rdbuf();

    // 解析
    json::value jv = json::parse(buffer.str());

    // 访问配置
    std::cout << "Database: " << jv.at("database").at("host").as_string() << std::endl;

    return 0;
}
```

---

## 访问 JSON 数据

### 安全访问

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    json::value jv = json::parse(R"({"name": "test", "age": 25})");

    // 1. 使用 at() - 抛出异常
    try {
        std::cout << jv.at("name").as_string() << std::endl;
        std::cout << jv.at("missing").as_string() << std::endl;  // 抛出异常
    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    // 2. 使用 if_contains() - 不抛异常
    if (json::value* v = jv.as_object().if_contains("name")) {
        std::cout << "姓名: " << v->as_string() << std::endl;
    }

    if (json::value* v = jv.as_object().if_contains("missing")) {
        std::cout << "找到 missing" << std::endl;
    } else {
        std::cout << "未找到 missing" << std::endl;
    }

    return 0;
}
```

### 类型检查

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

void print_value(const json::value& jv) {
    if (jv.is_null()) {
        std::cout << "null" << std::endl;
    } else if (jv.is_bool()) {
        std::cout << "bool: " << jv.as_bool() << std::endl;
    } else if (jv.is_int64()) {
        std::cout << "int64: " << jv.as_int64() << std::endl;
    } else if (jv.is_uint64()) {
        std::cout << "uint64: " << jv.as_uint64() << std::endl;
    } else if (jv.is_double()) {
        std::cout << "double: " << jv.as_double() << std::endl;
    } else if (jv.is_string()) {
        std::cout << "string: " << jv.as_string() << std::endl;
    } else if (jv.is_array()) {
        std::cout << "array with " << jv.as_array().size() << " elements" << std::endl;
    } else if (jv.is_object()) {
        std::cout << "object with " << jv.as_object().size() << " fields" << std::endl;
    }
}

int main() {
    print_value(nullptr);
    print_value(true);
    print_value(42);
    print_value(3.14);
    print_value("hello");
    print_value(json::array{1, 2, 3});
    print_value(json::object{{"key", "value"}});

    return 0;
}
```

---

## 序列化

### 基本序列化

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    json::object obj = {
        {"name", "Charlie"},
        {"age", 40},
        {"scores", {85, 90, 95}},
        {"metadata", {
            {"created", "2024-03-15"},
            {"updated", "2024-03-16"}
        }}
    };

    // 序列化（紧凑格式）
    std::string compact = json::serialize(obj);
    std::cout << "紧凑:\n" << compact << "\n\n";

    // 格式化输出
    std::cout << "格式化:\n" << std::setw(2) << obj << std::endl;

    return 0;
}
```

### 自定义序列化

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

struct Person {
    std::string name;
    int age;
    std::vector<std::string> hobbies;
};

// 自定义序列化
void tag_invoke(json::value_from_tag, json::value& jv, const Person& p) {
    jv = {
        {"name", p.name},
        {"age", p.age},
        {"hobbies", json::value_from(p.hobbies)}
    };
}

// 自定义反序列化
Person tag_invoke(json::value_to_tag<Person>, const json::value& jv) {
    Person p;
    p.name = jv.at("name").as_string().c_str();
    p.age = jv.at("age").as_int64();

    for (const auto& hobby : jv.at("hobbies").as_array()) {
        p.hobbies.push_back(hobby.as_string().c_str());
    }

    return p;
}

int main() {
    // 对象转 JSON
    Person person{"David", 32, {"gaming", "music"}};
    json::value jv = json::value_from(person);
    std::cout << jv << "\n\n";

    // JSON 转对象
    std::string json_str = R"({"name":"Eve","age":29,"hobbies":["art","travel"]})";
    Person person2 = json::value_to<Person>(json::parse(json_str));

    std::cout << "姓名: " << person2.name << std::endl;
    std::cout << "年龄: " << person2.age << std::endl;
    std::cout << "爱好: ";
    for (const auto& h : person2.hobbies) {
        std::cout << h << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 遍历 JSON

### 遍历数组

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    json::value jv = json::parse(R"([1, 2, 3, 4, 5])");

    // 方法1: 范围 for
    for (const auto& item : jv.as_array()) {
        std::cout << item.as_int64() << " ";
    }
    std::cout << "\n";

    // 方法2: 迭代器
    const json::array& arr = jv.as_array();
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << it->as_int64() << " ";
    }
    std::cout << "\n";

    return 0;
}
```

### 遍历对象

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    json::value jv = json::parse(R"({
        "name": "Frank",
        "age": 45,
        "city": "Shanghai"
    })");

    // 遍历所有键值对
    for (const auto& [key, value] : jv.as_object()) {
        std::cout << key << ": ";

        if (value.is_string()) {
            std::cout << value.as_string();
        } else if (value.is_int64()) {
            std::cout << value.as_int64();
        }

        std::cout << std::endl;
    }

    return 0;
}
```

---

## 实用示例

### 配置文件管理

```cpp
#include <boost/json.hpp>
#include <iostream>
#include <fstream>

namespace json = boost::json;

class Config {
public:
    bool load(const std::string& filename) {
        std::ifstream file(filename);
        if (!file) {
            return false;
        }

        std::stringstream buffer;
        buffer << file.rdbuf();

        try {
            config_ = json::parse(buffer.str());
            return true;
        } catch (const std::exception& e) {
            std::cerr << "解析错误: " << e.what() << std::endl;
            return false;
        }
    }

    template<typename T>
    T get(const std::string& path, const T& default_value) const {
        try {
            // 简化版：不支持嵌套路径
            return json::value_to<T>(config_.at(path));
        } catch (...) {
            return default_value;
        }
    }

private:
    json::value config_;
};

int main() {
    // config.json 内容:
    // {"app_name": "MyApp", "port": 8080, "debug": true}

    Config config;
    if (config.load("config.json")) {
        std::cout << "应用名称: " << config.get<std::string>("app_name", "Unknown") << std::endl;
        std::cout << "端口: " << config.get<int>("port", 0) << std::endl;
        std::cout << "调试模式: " << (config.get<bool>("debug", false) ? "是" : "否") << std::endl;
    }

    return 0;
}
```

### REST API 响应解析

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

struct User {
    int id;
    std::string username;
    std::string email;
};

std::vector<User> parse_users(const std::string& json_response) {
    std::vector<User> users;

    json::value jv = json::parse(json_response);

    for (const auto& item : jv.as_array()) {
        User user;
        user.id = item.at("id").as_int64();
        user.username = item.at("username").as_string().c_str();
        user.email = item.at("email").as_string().c_str();
        users.push_back(user);
    }

    return users;
}

int main() {
    std::string api_response = R"([
        {"id": 1, "username": "alice", "email": "alice@example.com"},
        {"id": 2, "username": "bob", "email": "bob@example.com"},
        {"id": 3, "username": "charlie", "email": "charlie@example.com"}
    ])";

    auto users = parse_users(api_response);

    for (const auto& user : users) {
        std::cout << "ID: " << user.id
                  << ", 用户名: " << user.username
                  << ", 邮箱: " << user.email
                  << std::endl;
    }

    return 0;
}
```

### 生成 JSON API

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

struct Product {
    int id;
    std::string name;
    double price;
    int stock;
};

std::string create_product_json(const Product& product) {
    json::object obj = {
        {"id", product.id},
        {"name", product.name},
        {"price", product.price},
        {"stock", product.stock},
        {"available", product.stock > 0}
    };

    return json::serialize(obj);
}

std::string create_products_list(const std::vector<Product>& products) {
    json::array arr;

    for (const auto& p : products) {
        arr.push_back(json::object{
            {"id", p.id},
            {"name", p.name},
            {"price", p.price},
            {"stock", p.stock}
        });
    }

    json::object response = {
        {"total", products.size()},
        {"products", arr}
    };

    return json::serialize(response);
}

int main() {
    Product p1{1, "Laptop", 999.99, 10};
    std::cout << create_product_json(p1) << "\n\n";

    std::vector<Product> products = {
        {1, "Laptop", 999.99, 10},
        {2, "Mouse", 29.99, 50},
        {3, "Keyboard", 79.99, 0}
    };

    std::cout << create_products_list(products) << std::endl;

    return 0;
}
```

---

## 错误处理

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    std::string invalid_json = R"({"name": "test", invalid})";

    try {
        json::value jv = json::parse(invalid_json);
    } catch (const json::system_error& e) {
        std::cerr << "解析错误: " << e.what() << std::endl;
        std::cerr << "错误码: " << e.code() << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "其他错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 性能优化

### 使用自定义分配器

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    // 使用自定义存储
    unsigned char buffer[4096];
    json::static_resource mr(buffer, sizeof(buffer));

    json::parse_options opts;
    opts.allow_comments = true;
    opts.allow_trailing_commas = true;

    json::value jv = json::parse(R"({"key": "value"})", &mr, opts);

    std::cout << jv << std::endl;

    return 0;
}
```

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_json -o example

# CMake
find_package(Boost REQUIRED COMPONENTS json)
target_link_libraries(myapp Boost::json)
```

---

## 参考资源

- [Boost.JSON 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/json/doc/html/index.html)

# Boost.PropertyTree - 属性树库

## 概述

Boost.PropertyTree 提供树形数据结构，支持多种格式（JSON、XML、INI）的读写。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>

namespace pt = boost::property_tree;

int main() {
    // 创建属性树
    pt::ptree tree;
    tree.put("name", "Alice");
    tree.put("age", 30);
    tree.put("city", "Beijing");

    // 输出 JSON
    pt::write_json(std::cout, tree);

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/property_tree/ptree.hpp>
#include <iostream>

namespace pt = boost::property_tree;

int main() {
    pt::ptree tree;

    // 设置值
    tree.put("server.host", "localhost");
    tree.put("server.port", 8080);
    tree.put("database.name", "mydb");
    tree.put("database.user", "admin");

    // 读取值
    std::string host = tree.get<std::string>("server.host");
    int port = tree.get<int>("server.port");
    std::string dbname = tree.get<std::string>("database.name");

    std::cout << "服务器: " << host << ":" << port << std::endl;
    std::cout << "数据库: " << dbname << std::endl;

    // 使用默认值
    int timeout = tree.get("server.timeout", 30);
    std::cout << "超时: " << timeout << "秒" << std::endl;

    return 0;
}
```

---

## JSON 读取

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    std::string json_data = R"({
        "name": "Alice",
        "age": 30,
        "email": "alice@example.com",
        "active": true
    })";

    pt::ptree tree;
    std::istringstream is(json_data);

    try {
        pt::read_json(is, tree);

        std::string name = tree.get<std::string>("name");
        int age = tree.get<int>("age");
        std::string email = tree.get<std::string>("email");
        bool active = tree.get<bool>("active");

        std::cout << "姓名: " << name << std::endl;
        std::cout << "年龄: " << age << std::endl;
        std::cout << "邮箱: " << email << std::endl;
        std::cout << "活跃: " << std::boolalpha << active << std::endl;

    } catch (const pt::json_parser_error& e) {
        std::cerr << "解析错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## JSON 写入

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>

namespace pt = boost::property_tree;

int main() {
    pt::ptree tree;

    tree.put("user.name", "Bob");
    tree.put("user.age", 25);
    tree.put("user.email", "bob@example.com");

    tree.put("settings.theme", "dark");
    tree.put("settings.notifications", true);
    tree.put("settings.language", "zh-CN");

    // 写入到标准输出
    pt::write_json(std::cout, tree);

    // 写入到文件
    // pt::write_json("config.json", tree);

    return 0;
}
```

---

## 数组处理

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    std::string json_data = R"({
        "users": [
            {"name": "Alice", "age": 30},
            {"name": "Bob", "age": 25},
            {"name": "Charlie", "age": 35}
        ]
    })";

    pt::ptree tree;
    std::istringstream is(json_data);
    pt::read_json(is, tree);

    std::cout << "用户列表:\n";
    for (const auto& item : tree.get_child("users")) {
        std::string name = item.second.get<std::string>("name");
        int age = item.second.get<int>("age");
        std::cout << "  " << name << " (" << age << "岁)" << std::endl;
    }

    return 0;
}
```

---

## XML 操作

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/xml_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    std::string xml_data = R"(
    <config>
        <server>
            <host>localhost</host>
            <port>8080</port>
        </server>
        <database>
            <name>mydb</name>
            <user>admin</user>
        </database>
    </config>
    )";

    pt::ptree tree;
    std::istringstream is(xml_data);

    try {
        pt::read_xml(is, tree);

        std::string host = tree.get<std::string>("config.server.host");
        int port = tree.get<int>("config.server.port");
        std::string dbname = tree.get<std::string>("config.database.name");

        std::cout << "服务器: " << host << ":" << port << std::endl;
        std::cout << "数据库: " << dbname << std::endl;

        // 写入 XML
        pt::write_xml(std::cout, tree);

    } catch (const pt::xml_parser_error& e) {
        std::cerr << "解析错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## INI 文件操作

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/ini_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    std::string ini_data = R"(
[server]
host=localhost
port=8080

[database]
name=mydb
user=admin
password=secret
    )";

    pt::ptree tree;
    std::istringstream is(ini_data);

    try {
        pt::read_ini(is, tree);

        std::string host = tree.get<std::string>("server.host");
        int port = tree.get<int>("server.port");
        std::string dbname = tree.get<std::string>("database.name");
        std::string user = tree.get<std::string>("database.user");

        std::cout << "服务器: " << host << ":" << port << std::endl;
        std::cout << "数据库: " << dbname << std::endl;
        std::cout << "用户: " << user << std::endl;

        // 写入 INI
        pt::write_ini(std::cout, tree);

    } catch (const pt::ini_parser_error& e) {
        std::cerr << "解析错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 嵌套结构

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>

namespace pt = boost::property_tree;

int main() {
    pt::ptree tree;

    // 创建嵌套结构
    pt::ptree address;
    address.put("street", "123 Main St");
    address.put("city", "Beijing");
    address.put("zipcode", "100000");

    pt::ptree person;
    person.put("name", "Alice");
    person.put("age", 30);
    person.add_child("address", address);

    tree.add_child("person", person);

    // 输出
    pt::write_json(std::cout, tree);

    return 0;
}
```

---

## 遍历树

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

void print_tree(const pt::ptree& tree, int level = 0) {
    std::string indent(level * 2, ' ');

    for (const auto& item : tree) {
        std::cout << indent << item.first << ": ";

        if (item.second.empty()) {
            std::cout << item.second.data() << std::endl;
        } else {
            std::cout << std::endl;
            print_tree(item.second, level + 1);
        }
    }
}

int main() {
    std::string json_data = R"({
        "server": {
            "host": "localhost",
            "port": 8080
        },
        "database": {
            "name": "mydb",
            "user": "admin"
        }
    })";

    pt::ptree tree;
    std::istringstream is(json_data);
    pt::read_json(is, tree);

    std::cout << "树结构:\n";
    print_tree(tree);

    return 0;
}
```

---

## 参考资源

- [Boost.PropertyTree 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/property_tree.html)

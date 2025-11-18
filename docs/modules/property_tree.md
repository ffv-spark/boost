# Boost.PropertyTree - 属性树库

## 概述

Boost.PropertyTree 提供用于处理分层键值数据的树形数据结构，支持多种格式（JSON、XML、INI）。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <string>

namespace pt = boost::property_tree;

int main() {
    pt::ptree tree;

    // 设置值
    tree.put("name", "John Doe");
    tree.put("age", 30);
    tree.put("address.city", "New York");
    tree.put("address.zip", "10001");

    // 读取值
    std::string name = tree.get<std::string>("name");
    int age = tree.get<int>("age");
    std::string city = tree.get<std::string>("address.city");

    std::cout << name << ", " << age << ", " << city << std::endl;

    // 写入 JSON 文件
    pt::write_json("config.json", tree);

    return 0;
}
```

---

## JSON 处理

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    // 创建 JSON 数据
    pt::ptree root;
    root.put("server.host", "localhost");
    root.put("server.port", 8080);
    root.put("server.ssl", true);

    // 添加数组
    pt::ptree features;
    pt::ptree feature1, feature2, feature3;
    feature1.put("", "logging");
    feature2.put("", "authentication");
    feature3.put("", "caching");

    features.push_back(std::make_pair("", feature1));
    features.push_back(std::make_pair("", feature2));
    features.push_back(std::make_pair("", feature3));

    root.add_child("features", features);

    // 写入 JSON 字符串
    std::stringstream ss;
    pt::write_json(ss, root);
    std::cout << "JSON output:\n" << ss.str() << std::endl;

    // 从 JSON 字符串读取
    pt::ptree loaded;
    std::stringstream input(ss.str());
    pt::read_json(input, loaded);

    // 读取值
    std::string host = loaded.get<std::string>("server.host");
    int port = loaded.get<int>("server.port");

    std::cout << "\nLoaded: " << host << ":" << port << std::endl;

    // 遍历数组
    std::cout << "Features:\n";
    for (const auto& item : loaded.get_child("features")) {
        std::cout << "  - " << item.second.get_value<std::string>() << std::endl;
    }

    return 0;
}
```

---

## XML 处理

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/xml_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    // 创建 XML 数据
    pt::ptree root;
    root.put("config.database.host", "localhost");
    root.put("config.database.port", 5432);
    root.put("config.database.name", "mydb");

    // 添加属性
    root.put("config.database.<xmlattr>.type", "postgresql");

    // 添加多个子节点
    pt::ptree& users = root.add("config.users", "");

    pt::ptree user1;
    user1.put("<xmlattr>.id", 1);
    user1.put("name", "Alice");
    user1.put("role", "admin");
    users.add_child("user", user1);

    pt::ptree user2;
    user2.put("<xmlattr>.id", 2);
    user2.put("name", "Bob");
    user2.put("role", "user");
    users.add_child("user", user2);

    // 写入 XML
    std::stringstream ss;
    pt::write_xml(ss, root);
    std::cout << "XML output:\n" << ss.str() << std::endl;

    // 读取 XML
    pt::ptree loaded;
    std::stringstream input(ss.str());
    pt::read_xml(input, loaded);

    // 访问属性
    std::string db_type = loaded.get<std::string>("config.database.<xmlattr>.type");
    std::cout << "\nDatabase type: " << db_type << std::endl;

    // 遍历用户
    std::cout << "Users:\n";
    for (const auto& user : loaded.get_child("config.users")) {
        if (user.first == "user") {
            int id = user.second.get<int>("<xmlattr>.id");
            std::string name = user.second.get<std::string>("name");
            std::string role = user.second.get<std::string>("role");
            std::cout << "  ID: " << id << ", Name: " << name
                      << ", Role: " << role << std::endl;
        }
    }

    return 0;
}
```

---

## INI 文件处理

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/ini_parser.hpp>
#include <iostream>
#include <sstream>

namespace pt = boost::property_tree;

int main() {
    // 创建 INI 数据
    pt::ptree ini;

    ini.put("General.app_name", "MyApp");
    ini.put("General.version", "1.0.0");

    ini.put("Database.host", "localhost");
    ini.put("Database.port", "5432");
    ini.put("Database.username", "admin");

    ini.put("Logging.level", "INFO");
    ini.put("Logging.file", "/var/log/myapp.log");

    // 写入 INI 文件
    std::stringstream ss;
    pt::write_ini(ss, ini);
    std::cout << "INI output:\n" << ss.str() << std::endl;

    // 读取 INI
    pt::ptree loaded;
    std::stringstream input(ss.str());
    pt::read_ini(input, loaded);

    // 读取值
    std::string app_name = loaded.get<std::string>("General.app_name");
    std::string db_host = loaded.get<std::string>("Database.host");

    std::cout << "\nApp: " << app_name << std::endl;
    std::cout << "Database: " << db_host << std::endl;

    return 0;
}
```

---

## 配置文件管理

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <string>

namespace pt = boost::property_tree;

class ConfigManager {
public:
    ConfigManager(const std::string& filename) : filename_(filename) {
        try {
            pt::read_json(filename, config_);
        } catch (const pt::json_parser_error& e) {
            std::cerr << "Error loading config: " << e.what() << std::endl;
            // 使用默认配置
            set_defaults();
        }
    }

    template<typename T>
    T get(const std::string& key, const T& default_value) const {
        return config_.get(key, default_value);
    }

    template<typename T>
    void set(const std::string& key, const T& value) {
        config_.put(key, value);
    }

    void save() {
        try {
            pt::write_json(filename_, config_);
        } catch (const pt::json_parser_error& e) {
            std::cerr << "Error saving config: " << e.what() << std::endl;
        }
    }

    void print() const {
        pt::write_json(std::cout, config_);
    }

private:
    void set_defaults() {
        config_.put("app.name", "DefaultApp");
        config_.put("app.version", "1.0.0");
        config_.put("server.port", 8080);
        config_.put("server.threads", 4);
    }

    std::string filename_;
    pt::ptree config_;
};

int main() {
    ConfigManager config("app_config.json");

    // 读取配置
    std::string app_name = config.get<std::string>("app.name", "Unknown");
    int port = config.get<int>("server.port", 8080);

    std::cout << "App: " << app_name << std::endl;
    std::cout << "Port: " << port << std::endl;

    // 修改配置
    config.set("server.port", 9090);
    config.set("server.ssl", true);

    // 保存
    config.save();

    std::cout << "\nCurrent configuration:\n";
    config.print();

    return 0;
}
```

---

## 嵌套数据访问

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <optional>

namespace pt = boost::property_tree;

int main() {
    pt::ptree tree;

    // 创建深层嵌套结构
    tree.put("company.name", "TechCorp");
    tree.put("company.employees.engineering.count", 50);
    tree.put("company.employees.engineering.manager", "Alice");
    tree.put("company.employees.sales.count", 30);
    tree.put("company.employees.sales.manager", "Bob");

    // 安全访问（使用 get_optional）
    auto manager = tree.get_optional<std::string>("company.employees.engineering.manager");
    if (manager) {
        std::cout << "Engineering Manager: " << *manager << std::endl;
    }

    // 访问不存在的键
    auto missing = tree.get_optional<std::string>("company.employees.hr.manager");
    if (!missing) {
        std::cout << "HR manager not found" << std::endl;
    }

    // 获取子树
    try {
        pt::ptree& engineering = tree.get_child("company.employees.engineering");
        std::cout << "\nEngineering department:\n";
        for (const auto& item : engineering) {
            std::cout << "  " << item.first << ": "
                      << item.second.get_value<std::string>() << std::endl;
        }
    } catch (const pt::ptree_bad_path& e) {
        std::cerr << "Path not found: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 数组处理技巧

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <vector>

namespace pt = boost::property_tree;

struct Person {
    std::string name;
    int age;
    std::vector<std::string> hobbies;
};

int main() {
    pt::ptree root;

    // 创建人员数组
    pt::ptree people_array;

    // 添加第一个人
    pt::ptree person1;
    person1.put("name", "Alice");
    person1.put("age", 30);

    pt::ptree hobbies1;
    pt::ptree hobby1, hobby2;
    hobby1.put("", "reading");
    hobby2.put("", "hiking");
    hobbies1.push_back(std::make_pair("", hobby1));
    hobbies1.push_back(std::make_pair("", hobby2));
    person1.add_child("hobbies", hobbies1);

    people_array.push_back(std::make_pair("", person1));

    // 添加第二个人
    pt::ptree person2;
    person2.put("name", "Bob");
    person2.put("age", 25);

    pt::ptree hobbies2;
    pt::ptree hobby3;
    hobby3.put("", "gaming");
    hobbies2.push_back(std::make_pair("", hobby3));
    person2.add_child("hobbies", hobbies2);

    people_array.push_back(std::make_pair("", person2));

    root.add_child("people", people_array);

    // 输出 JSON
    std::cout << "JSON output:\n";
    pt::write_json(std::cout, root);

    // 解析数组
    std::cout << "\nParsed people:\n";
    for (const auto& person_item : root.get_child("people")) {
        const pt::ptree& person = person_item.second;

        std::string name = person.get<std::string>("name");
        int age = person.get<int>("age");

        std::cout << name << " (" << age << "), hobbies: ";

        for (const auto& hobby : person.get_child("hobbies")) {
            std::cout << hobby.second.get_value<std::string>() << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.PropertyTree 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/property_tree.html)

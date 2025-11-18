# Boost.UUID - 通用唯一标识符库

## 概述

Boost.UUID 提供通用唯一标识符（UUID）的生成和操作功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    // 生成随机 UUID
    boost::uuids::random_generator gen;
    boost::uuids::uuid id = gen();

    std::cout << "UUID: " << id << std::endl;

    return 0;
}
```

---

## UUID 基本操作

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    boost::uuids::uuid id;

    // 检查是否为 nil
    std::cout << std::boolalpha;
    std::cout << "是否为 nil: " << id.is_nil() << std::endl;

    // 获取大小
    std::cout << "大小: " << id.size() << " 字节" << std::endl;

    // 获取版本
    std::cout << "版本: " << (int)id.version() << std::endl;

    return 0;
}
```

---

## 随机 UUID

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    boost::uuids::random_generator gen;

    std::cout << "生成5个随机 UUID:\n";
    for (int i = 0; i < 5; ++i) {
        boost::uuids::uuid id = gen();
        std::cout << "  " << id << std::endl;
    }

    return 0;
}
```

---

## 基于名称的 UUID

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <string>

int main() {
    // DNS 命名空间 UUID
    boost::uuids::uuid dns_namespace = 
        boost::uuids::string_generator()("6ba7b810-9dad-11d1-80b4-00c04fd430c8");

    // 基于名称生成 UUID
    boost::uuids::name_generator gen(dns_namespace);

    std::string name1 = "example.com";
    std::string name2 = "google.com";

    boost::uuids::uuid id1 = gen(name1);
    boost::uuids::uuid id2 = gen(name2);

    std::cout << "example.com UUID: " << id1 << std::endl;
    std::cout << "google.com UUID: " << id2 << std::endl;

    // 相同名称生成相同 UUID
    boost::uuids::uuid id3 = gen(name1);
    std::cout << "再次生成 example.com: " << id3 << std::endl;
    std::cout << "id1 == id3: " << (id1 == id3) << std::endl;

    return 0;
}
```

---

## 字符串转换

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <sstream>

int main() {
    boost::uuids::random_generator gen;
    boost::uuids::uuid id = gen();

    // UUID 转字符串
    std::string uuid_str = boost::uuids::to_string(id);
    std::cout << "UUID 字符串: " << uuid_str << std::endl;

    // 字符串转 UUID
    boost::uuids::string_generator str_gen;
    boost::uuids::uuid id2 = str_gen(uuid_str);

    std::cout << "解析的 UUID: " << id2 << std::endl;
    std::cout << "相等: " << (id == id2) << std::endl;

    return 0;
}
```

---

## UUID 比较

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    boost::uuids::random_generator gen;

    boost::uuids::uuid id1 = gen();
    boost::uuids::uuid id2 = gen();
    boost::uuids::uuid id3 = id1;

    std::cout << std::boolalpha;
    std::cout << "id1: " << id1 << std::endl;
    std::cout << "id2: " << id2 << std::endl;
    std::cout << "id3: " << id3 << std::endl;

    std::cout << "\nid1 == id2: " << (id1 == id2) << std::endl;
    std::cout << "id1 == id3: " << (id1 == id3) << std::endl;
    std::cout << "id1 < id2: " << (id1 < id2) << std::endl;

    return 0;
}
```

---

## 容器中使用

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <vector>
#include <set>
#include <map>

int main() {
    boost::uuids::random_generator gen;

    // vector
    std::vector<boost::uuids::uuid> vec;
    for (int i = 0; i < 5; ++i) {
        vec.push_back(gen());
    }

    std::cout << "Vector 中的 UUID:\n";
    for (const auto& id : vec) {
        std::cout << "  " << id << std::endl;
    }

    // set（自动排序）
    std::set<boost::uuids::uuid> s;
    for (int i = 0; i < 5; ++i) {
        s.insert(gen());
    }

    std::cout << "\nSet 中的 UUID（已排序）:\n";
    for (const auto& id : s) {
        std::cout << "  " << id << std::endl;
    }

    // map
    std::map<boost::uuids::uuid, std::string> m;
    m[gen()] = "Alice";
    m[gen()] = "Bob";
    m[gen()] = "Charlie";

    std::cout << "\nMap 中的 UUID:\n";
    for (const auto& pair : m) {
        std::cout << "  " << pair.first << " -> " << pair.second << std::endl;
    }

    return 0;
}
```

---

## 数据库主键

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <string>
#include <map>

struct User {
    boost::uuids::uuid id;
    std::string name;
    std::string email;

    User(const std::string& n, const std::string& e)
        : name(n), email(e) {
        boost::uuids::random_generator gen;
        id = gen();
    }
};

int main() {
    std::map<boost::uuids::uuid, User> users;

    // 创建用户
    User alice("Alice", "alice@example.com");
    User bob("Bob", "bob@example.com");
    User charlie("Charlie", "charlie@example.com");

    users[alice.id] = alice;
    users[bob.id] = bob;
    users[charlie.id] = charlie;

    std::cout << "用户数据库:\n";
    for (const auto& pair : users) {
        std::cout << "  ID: " << pair.first << std::endl;
        std::cout << "    姓名: " << pair.second.name << std::endl;
        std::cout << "    邮箱: " << pair.second.email << std::endl;
    }

    return 0;
}
```

---

## Nil UUID

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    // Nil UUID (全零)
    boost::uuids::uuid nil_id = boost::uuids::nil_uuid();

    std::cout << "Nil UUID: " << nil_id << std::endl;
    std::cout << "是否为 nil: " << nil_id.is_nil() << std::endl;

    // 默认构造也是 nil
    boost::uuids::uuid default_id;
    std::cout << "默认 UUID: " << default_id << std::endl;
    std::cout << "是否为 nil: " << default_id.is_nil() << std::endl;

    return 0;
}
```

---

## 会话跟踪

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <string>
#include <map>
#include <chrono>

struct Session {
    boost::uuids::uuid id;
    std::string username;
    std::chrono::system_clock::time_point created;

    Session(const std::string& user) 
        : username(user), created(std::chrono::system_clock::now()) {
        boost::uuids::random_generator gen;
        id = gen();
    }
};

class SessionManager {
public:
    boost::uuids::uuid create_session(const std::string& username) {
        Session session(username);
        sessions_[session.id] = session;
        std::cout << "创建会话: " << session.id << " 用户: " << username << std::endl;
        return session.id;
    }

    bool validate_session(const boost::uuids::uuid& session_id) {
        return sessions_.find(session_id) != sessions_.end();
    }

    void close_session(const boost::uuids::uuid& session_id) {
        sessions_.erase(session_id);
        std::cout << "关闭会话: " << session_id << std::endl;
    }

private:
    std::map<boost::uuids::uuid, Session> sessions_;
};

int main() {
    SessionManager manager;

    // 创建会话
    auto session1 = manager.create_session("Alice");
    auto session2 = manager.create_session("Bob");

    // 验证会话
    std::cout << "\n验证会话:\n";
    std::cout << "session1 有效: " << manager.validate_session(session1) << std::endl;
    std::cout << "session2 有效: " << manager.validate_session(session2) << std::endl;

    // 关闭会话
    std::cout << std::endl;
    manager.close_session(session1);

    std::cout << "\n关闭后验证:\n";
    std::cout << "session1 有效: " << manager.validate_session(session1) << std::endl;
    std::cout << "session2 有效: " << manager.validate_session(session2) << std::endl;

    return 0;
}
```

---

## 哈希支持

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <boost/uuid/uuid_hash.hpp>
#include <iostream>
#include <unordered_map>
#include <unordered_set>

int main() {
    boost::uuids::random_generator gen;

    // unordered_set
    std::unordered_set<boost::uuids::uuid, boost::hash<boost::uuids::uuid>> s;
    for (int i = 0; i < 5; ++i) {
        s.insert(gen());
    }

    std::cout << "Unordered Set:\n";
    for (const auto& id : s) {
        std::cout << "  " << id << std::endl;
    }

    // unordered_map
    std::unordered_map<boost::uuids::uuid, std::string, boost::hash<boost::uuids::uuid>> m;
    m[gen()] = "Value 1";
    m[gen()] = "Value 2";
    m[gen()] = "Value 3";

    std::cout << "\nUnordered Map:\n";
    for (const auto& pair : m) {
        std::cout << "  " << pair.first << " -> " << pair.second << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.UUID 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/uuid/doc/html/index.html)

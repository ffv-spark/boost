# Boost.Uuid - UUID 生成库

## 概述

Boost.Uuid 提供通用唯一识别码（UUID）的生成和操作。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    // 随机 UUID
    boost::uuids::random_generator gen;
    boost::uuids::uuid id = gen();

    std::cout << "UUID: " << id << std::endl;

    return 0;
}
```

---

## UUID 生成器

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    // 1. 随机生成器（最常用）
    boost::uuids::random_generator random_gen;
    auto uuid1 = random_gen();
    std::cout << "Random: " << uuid1 << std::endl;

    // 2. 名称生成器（基于命名空间和名称）
    boost::uuids::name_generator name_gen(
        boost::uuids::ns::dns());
    auto uuid2 = name_gen("example.com");
    std::cout << "Name-based: " << uuid2 << std::endl;

    // 3. 空 UUID
    boost::uuids::nil_generator nil_gen;
    auto uuid3 = nil_gen();
    std::cout << "Nil: " << uuid3 << std::endl;

    // 4. 字符串生成器
    boost::uuids::string_generator string_gen;
    auto uuid4 = string_gen("01234567-89ab-cdef-0123-456789abcdef");
    std::cout << "From string: " << uuid4 << std::endl;

    return 0;
}
```

---

## UUID 操作

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <vector>

int main() {
    boost::uuids::random_generator gen;
    boost::uuids::uuid id1 = gen();
    boost::uuids::uuid id2 = gen();

    // 比较
    std::cout << "Equal: " << (id1 == id2) << std::endl;
    std::cout << "Not equal: " << (id1 != id2) << std::endl;
    std::cout << "Less than: " << (id1 < id2) << std::endl;

    // 转字符串
    std::string str = boost::uuids::to_string(id1);
    std::cout << "String: " << str << std::endl;

    // 获取字节
    std::cout << "Size: " << id1.size() << " bytes" << std::endl;

    std::cout << "Bytes: ";
    for (auto byte : id1) {
        printf("%02x ", byte);
    }
    std::cout << std::endl;

    // 判空
    boost::uuids::uuid nil;
    std::cout << "Is nil: " << nil.is_nil() << std::endl;

    return 0;
}
```

---

## 实用示例

### 数据库主键

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <map>
#include <string>

struct User {
    boost::uuids::uuid id;
    std::string name;
    int age;
};

class UserDatabase {
public:
    UserDatabase() : gen_() {}

    boost::uuids::uuid create_user(const std::string& name, int age) {
        User user;
        user.id = gen_();
        user.name = name;
        user.age = age;

        users_[user.id] = user;

        std::cout << "Created user " << user.id << ": " << name << std::endl;

        return user.id;
    }

    User* get_user(const boost::uuids::uuid& id) {
        auto it = users_.find(id);
        if (it != users_.end()) {
            return &it->second;
        }
        return nullptr;
    }

private:
    boost::uuids::random_generator gen_;
    std::map<boost::uuids::uuid, User> users_;
};

int main() {
    UserDatabase db;

    auto id1 = db.create_user("Alice", 25);
    auto id2 = db.create_user("Bob", 30);

    if (User* user = db.get_user(id1)) {
        std::cout << "Found: " << user->name << ", age " << user->age << std::endl;
    }

    return 0;
}
```

### 文件唯一命名

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <fstream>

std::string generate_filename(const std::string& prefix, const std::string& ext) {
    boost::uuids::random_generator gen;
    boost::uuids::uuid id = gen();

    return prefix + "_" + boost::uuids::to_string(id) + ext;
}

int main() {
    std::string filename = generate_filename("upload", ".jpg");
    std::cout << "Generated filename: " << filename << std::endl;

    // 创建文件
    std::ofstream file(filename);
    file << "File content";
    file.close();

    std::cout << "File created: " << filename << std::endl;

    return 0;
}
```

### 会话管理

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>
#include <map>
#include <string>
#include <ctime>

struct Session {
    boost::uuids::uuid id;
    std::string user_id;
    std::time_t created_at;
    std::time_t last_accessed;
};

class SessionManager {
public:
    boost::uuids::uuid create_session(const std::string& user_id) {
        Session session;
        session.id = gen_();
        session.user_id = user_id;
        session.created_at = std::time(nullptr);
        session.last_accessed = session.created_at;

        sessions_[session.id] = session;

        std::cout << "Created session " << session.id
                  << " for user " << user_id << std::endl;

        return session.id;
    }

    bool validate_session(const boost::uuids::uuid& session_id) {
        auto it = sessions_.find(session_id);
        if (it != sessions_.end()) {
            it->second.last_accessed = std::time(nullptr);
            return true;
        }
        return false;
    }

    void cleanup_expired_sessions(int timeout_seconds) {
        std::time_t now = std::time(nullptr);
        for (auto it = sessions_.begin(); it != sessions_.end(); ) {
            if (now - it->second.last_accessed > timeout_seconds) {
                std::cout << "Removing expired session " << it->first << std::endl;
                it = sessions_.erase(it);
            } else {
                ++it;
            }
        }
    }

private:
    boost::uuids::random_generator gen_;
    std::map<boost::uuids::uuid, Session> sessions_;
};

int main() {
    SessionManager manager;

    auto sid1 = manager.create_session("user1");
    auto sid2 = manager.create_session("user2");

    std::cout << "\nValidating sessions:\n";
    std::cout << "Session 1 valid: " << manager.validate_session(sid1) << std::endl;

    boost::uuids::uuid fake_id = boost::uuids::random_generator()();
    std::cout << "Fake session valid: " << manager.validate_session(fake_id) << std::endl;

    return 0;
}
```

---

## 名称空间

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <iostream>

int main() {
    // 预定义的命名空间
    boost::uuids::name_generator dns_gen(boost::uuids::ns::dns());
    boost::uuids::name_generator url_gen(boost::uuids::ns::url());
    boost::uuids::name_generator oid_gen(boost::uuids::ns::oid());
    boost::uuids::name_generator x500_gen(boost::uuids::ns::x500());

    // 基于域名生成 UUID
    auto dns_uuid = dns_gen("example.com");
    std::cout << "DNS UUID: " << dns_uuid << std::endl;

    // 同样的输入总是产生同样的 UUID
    auto dns_uuid2 = dns_gen("example.com");
    std::cout << "Same input: " << (dns_uuid == dns_uuid2) << std::endl;

    // 不同输入产生不同 UUID
    auto dns_uuid3 = dns_gen("other.com");
    std::cout << "Different input: " << (dns_uuid != dns_uuid3) << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **随机生成**: 大多数情况使用 random_generator
2. **名称生成**: 需要可重现性时使用 name_generator
3. **存储**: UUID 占用 16 字节
4. **性能**: 生成速度快
5. **唯一性**: 随机 UUID 冲突概率极低

---

## 参考资源

- [Boost.Uuid 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/uuid/doc/uuid.html)
- [RFC 4122](https://tools.ietf.org/html/rfc4122)

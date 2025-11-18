# Boost.MultiIndex - 多索引容器库

## 概述

Boost.MultiIndex 提供具有多个索引的容器，可以从不同角度访问同一数据集。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Person {
    int id;
    std::string name;
    int age;
};

typedef multi_index_container<
    Person,
    indexed_by<
        ordered_unique<member<Person, int, &Person::id>>
    >
> PersonSet;

int main() {
    PersonSet persons;

    persons.insert({1, "Alice", 30});
    persons.insert({2, "Bob", 25});
    persons.insert({3, "Charlie", 35});

    for (const auto& p : persons) {
        std::cout << p.id << ": " << p.name << " (" << p.age << ")" << std::endl;
    }

    return 0;
}
```

---

## 多个索引

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Employee {
    int id;
    std::string name;
    double salary;
};

// 定义多索引容器
typedef multi_index_container<
    Employee,
    indexed_by<
        // 按 ID 排序（唯一）
        ordered_unique<member<Employee, int, &Employee::id>>,
        // 按姓名排序
        ordered_non_unique<member<Employee, std::string, &Employee::name>>,
        // 按薪水排序
        ordered_non_unique<member<Employee, double, &Employee::salary>>
    >
> EmployeeSet;

int main() {
    EmployeeSet employees;

    employees.insert({1, "Alice", 50000});
    employees.insert({2, "Bob", 45000});
    employees.insert({3, "Charlie", 60000});
    employees.insert({4, "David", 45000});

    // 按 ID 访问（索引0）
    std::cout << "按 ID:\n";
    auto& by_id = employees.get<0>();
    for (const auto& e : by_id) {
        std::cout << "  " << e.id << ": " << e.name << std::endl;
    }

    // 按姓名访问（索引1）
    std::cout << "\n按姓名:\n";
    auto& by_name = employees.get<1>();
    for (const auto& e : by_name) {
        std::cout << "  " << e.name << ": $" << e.salary << std::endl;
    }

    // 按薪水访问（索引2）
    std::cout << "\n按薪水:\n";
    auto& by_salary = employees.get<2>();
    for (const auto& e : by_salary) {
        std::cout << "  $" << e.salary << ": " << e.name << std::endl;
    }

    return 0;
}
```

---

## 查找操作

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Book {
    int isbn;
    std::string title;
    std::string author;
};

typedef multi_index_container<
    Book,
    indexed_by<
        ordered_unique<member<Book, int, &Book::isbn>>,
        ordered_non_unique<member<Book, std::string, &Book::author>>
    >
> BookSet;

int main() {
    BookSet books;

    books.insert({12345, "C++ Primer", "Lippman"});
    books.insert({67890, "Effective C++", "Meyers"});
    books.insert({11111, "More Effective C++", "Meyers"});

    // 按 ISBN 查找
    auto& by_isbn = books.get<0>();
    auto it = by_isbn.find(67890);
    if (it != by_isbn.end()) {
        std::cout << "找到书籍: " << it->title << std::endl;
    }

    // 按作者查找
    auto& by_author = books.get<1>();
    auto range = by_author.equal_range("Meyers");

    std::cout << "\nMeyers 的书:\n";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->title << std::endl;
    }

    return 0;
}
```

---

## 修改元素

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Product {
    int id;
    std::string name;
    double price;
};

typedef multi_index_container<
    Product,
    indexed_by<
        ordered_unique<member<Product, int, &Product::id>>
    >
> ProductSet;

int main() {
    ProductSet products;

    products.insert({1, "Apple", 1.99});
    products.insert({2, "Banana", 0.99});
    products.insert({3, "Orange", 2.49});

    std::cout << "原始价格:\n";
    for (const auto& p : products) {
        std::cout << "  " << p.name << ": $" << p.price << std::endl;
    }

    // 修改元素
    auto it = products.find(2);
    if (it != products.end()) {
        products.modify(it, [](Product& p) {
            p.price = 1.29;  // 香蕉涨价
        });
    }

    std::cout << "\n修改后:\n";
    for (const auto& p : products) {
        std::cout << "  " << p.name << ": $" << p.price << std::endl;
    }

    return 0;
}
```

---

## 哈希索引

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/hashed_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct User {
    int id;
    std::string email;
    std::string username;
};

typedef multi_index_container<
    User,
    indexed_by<
        hashed_unique<member<User, int, &User::id>>,
        hashed_unique<member<User, std::string, &User::email>>
    >
> UserSet;

int main() {
    UserSet users;

    users.insert({1, "alice@example.com", "alice"});
    users.insert({2, "bob@example.com", "bob"});
    users.insert({3, "charlie@example.com", "charlie"});

    // 按 ID 查找（哈希索引0）
    auto& by_id = users.get<0>();
    auto it1 = by_id.find(2);
    if (it1 != by_id.end()) {
        std::cout << "用户 ID 2: " << it1->username << std::endl;
    }

    // 按邮箱查找（哈希索引1）
    auto& by_email = users.get<1>();
    auto it2 = by_email.find("charlie@example.com");
    if (it2 != by_email.end()) {
        std::cout << "charlie@example.com: " << it2->username << std::endl;
    }

    return 0;
}
```

---

## 复合键

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/composite_key.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Record {
    std::string category;
    std::string name;
    int value;
};

typedef multi_index_container<
    Record,
    indexed_by<
        ordered_non_unique<
            composite_key<
                Record,
                member<Record, std::string, &Record::category>,
                member<Record, std::string, &Record::name>
            >
        >
    >
> RecordSet;

int main() {
    RecordSet records;

    records.insert({"A", "Item1", 10});
    records.insert({"A", "Item2", 20});
    records.insert({"B", "Item1", 30});
    records.insert({"B", "Item3", 40});

    // 查找特定类别和名称
    auto& idx = records.get<0>();
    auto it = idx.find(boost::make_tuple("A", "Item2"));

    if (it != idx.end()) {
        std::cout << "找到: " << it->category << " - " 
                  << it->name << ": " << it->value << std::endl;
    }

    // 查找某个类别的所有记录
    auto range = idx.equal_range(boost::make_tuple("B"));
    std::cout << "\n类别 B 的记录:\n";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->name << ": " << it->value << std::endl;
    }

    return 0;
}
```

---

## 序列索引

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/sequenced_index.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Task {
    int id;
    std::string description;
};

typedef multi_index_container<
    Task,
    indexed_by<
        sequenced<>,  // 插入顺序
        ordered_unique<member<Task, int, &Task::id>>
    >
> TaskList;

int main() {
    TaskList tasks;

    tasks.push_back({3, "任务3"});
    tasks.push_back({1, "任务1"});
    tasks.push_back({2, "任务2"});

    // 按插入顺序遍历
    std::cout << "插入顺序:\n";
    auto& seq = tasks.get<0>();
    for (const auto& t : seq) {
        std::cout << "  " << t.id << ": " << t.description << std::endl;
    }

    // 按 ID 排序遍历
    std::cout << "\n按 ID 排序:\n";
    auto& by_id = tasks.get<1>();
    for (const auto& t : by_id) {
        std::cout << "  " << t.id << ": " << t.description << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.MultiIndex 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/multi_index/doc/index.html)

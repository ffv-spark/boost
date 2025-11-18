# Boost.Multi-index - 多索引容器库

## 概述

Boost.Multi-index 提供支持多个索引的容器，允许以不同方式访问相同的数据。

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

struct Employee {
    int id;
    std::string name;
    int age;

    Employee(int i, const std::string& n, int a) : id(i), name(n), age(a) {}
};

// 定义多索引容器
typedef multi_index_container<
    Employee,
    indexed_by<
        // 按 ID 排序的索引
        ordered_unique<member<Employee, int, &Employee::id>>,
        // 按名字排序的索引
        ordered_non_unique<member<Employee, std::string, &Employee::name>>
    >
> EmployeeSet;

int main() {
    EmployeeSet employees;

    employees.insert(Employee(1, "Alice", 30));
    employees.insert(Employee(2, "Bob", 25));
    employees.insert(Employee(3, "Charlie", 35));

    // 通过第一个索引（ID）访问
    auto& id_index = employees.get<0>();
    auto it = id_index.find(2);
    if (it != id_index.end()) {
        std::cout << "Found: " << it->name << ", age " << it->age << std::endl;
    }

    // 通过第二个索引（名字）访问
    auto& name_index = employees.get<1>();
    std::cout << "\nAll employees by name:\n";
    for (const auto& emp : name_index) {
        std::cout << emp.name << " (ID: " << emp.id << ")\n";
    }

    return 0;
}
```

---

## 多个排序索引

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

    Product(int i, const std::string& n, double p)
        : id(i), name(n), price(p) {}
};

// 按 ID、名字、价格三种方式索引
typedef multi_index_container<
    Product,
    indexed_by<
        ordered_unique<member<Product, int, &Product::id>>,
        ordered_non_unique<member<Product, std::string, &Product::name>>,
        ordered_non_unique<member<Product, double, &Product::price>>
    >
> ProductCatalog;

// 为索引命名
struct by_id {};
struct by_name {};
struct by_price {};

typedef multi_index_container<
    Product,
    indexed_by<
        ordered_unique<tag<by_id>, member<Product, int, &Product::id>>,
        ordered_non_unique<tag<by_name>, member<Product, std::string, &Product::name>>,
        ordered_non_unique<tag<by_price>, member<Product, double, &Product::price>>
    >
> NamedProductCatalog;

int main() {
    NamedProductCatalog catalog;

    catalog.insert(Product(1, "Laptop", 999.99));
    catalog.insert(Product(2, "Mouse", 29.99));
    catalog.insert(Product(3, "Keyboard", 79.99));
    catalog.insert(Product(4, "Monitor", 299.99));

    // 按价格排序查看
    std::cout << "Products by price:\n";
    auto& price_index = catalog.get<by_price>();
    for (const auto& p : price_index) {
        std::cout << p.name << ": $" << p.price << std::endl;
    }

    // 按名字查找
    auto& name_index = catalog.get<by_name>();
    auto it = name_index.find("Mouse");
    if (it != name_index.end()) {
        std::cout << "\nFound: " << it->name << " (ID: " << it->id << ")\n";
    }

    return 0;
}
```

---

## 哈希索引

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/hashed_index.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct User {
    std::string username;
    std::string email;
    int age;

    User(const std::string& u, const std::string& e, int a)
        : username(u), email(e), age(a) {}
};

struct by_username {};
struct by_email {};
struct by_age {};

typedef multi_index_container<
    User,
    indexed_by<
        // 哈希索引用于快速查找
        hashed_unique<tag<by_username>, member<User, std::string, &User::username>>,
        hashed_unique<tag<by_email>, member<User, std::string, &User::email>>,
        // 排序索引用于范围查询
        ordered_non_unique<tag<by_age>, member<User, int, &User::age>>
    >
> UserDatabase;

int main() {
    UserDatabase db;

    db.insert(User("alice", "alice@example.com", 30));
    db.insert(User("bob", "bob@example.com", 25));
    db.insert(User("charlie", "charlie@example.com", 35));

    // 通过用户名快速查找（O(1)）
    auto& username_idx = db.get<by_username>();
    auto it = username_idx.find("bob");
    if (it != username_idx.end()) {
        std::cout << "User: " << it->username << ", Email: " << it->email << std::endl;
    }

    // 通过年龄范围查询
    auto& age_idx = db.get<by_age>();
    auto range = age_idx.equal_range(25);
    std::cout << "\nUsers aged 25:\n";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->username << std::endl;
    }

    return 0;
}
```

---

## 复合键索引

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/composite_key.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Transaction {
    std::string customer;
    std::string product;
    double amount;
    std::string date;

    Transaction(const std::string& c, const std::string& p,
                double a, const std::string& d)
        : customer(c), product(p), amount(a), date(d) {}
};

struct by_customer_date {};
struct by_product_amount {};

typedef multi_index_container<
    Transaction,
    indexed_by<
        // 按客户和日期组合索引
        ordered_non_unique<
            tag<by_customer_date>,
            composite_key<
                Transaction,
                member<Transaction, std::string, &Transaction::customer>,
                member<Transaction, std::string, &Transaction::date>
            >
        >,
        // 按产品和金额组合索引
        ordered_non_unique<
            tag<by_product_amount>,
            composite_key<
                Transaction,
                member<Transaction, std::string, &Transaction::product>,
                member<Transaction, double, &Transaction::amount>
            >
        >
    >
> TransactionLog;

int main() {
    TransactionLog log;

    log.insert(Transaction("Alice", "Laptop", 999.99, "2024-01-15"));
    log.insert(Transaction("Bob", "Mouse", 29.99, "2024-01-16"));
    log.insert(Transaction("Alice", "Keyboard", 79.99, "2024-01-17"));
    log.insert(Transaction("Charlie", "Monitor", 299.99, "2024-01-18"));
    log.insert(Transaction("Alice", "Mouse", 29.99, "2024-01-20"));

    // 查找特定客户的所有交易
    auto& customer_idx = log.get<by_customer_date>();
    auto range = customer_idx.equal_range("Alice");

    std::cout << "Alice's transactions:\n";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->date << ": " << it->product
                  << " ($" << it->amount << ")\n";
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

struct Account {
    int id;
    std::string name;
    double balance;

    Account(int i, const std::string& n, double b)
        : id(i), name(n), balance(b) {}
};

struct by_id {};
struct by_balance {};

typedef multi_index_container<
    Account,
    indexed_by<
        ordered_unique<tag<by_id>, member<Account, int, &Account::id>>,
        ordered_non_unique<tag<by_balance>, member<Account, double, &Account::balance>>
    >
> AccountSet;

int main() {
    AccountSet accounts;

    accounts.insert(Account(1, "Alice", 1000.0));
    accounts.insert(Account(2, "Bob", 500.0));
    accounts.insert(Account(3, "Charlie", 1500.0));

    // 修改元素
    auto& id_idx = accounts.get<by_id>();
    auto it = id_idx.find(2);

    if (it != id_idx.end()) {
        // 方法1: 使用 modify
        id_idx.modify(it, [](Account& acc) {
            acc.balance += 100.0;  // 存入 100
        });

        std::cout << "Bob's new balance: $" << it->balance << std::endl;
    }

    // 按余额排序查看
    std::cout << "\nAccounts by balance:\n";
    auto& balance_idx = accounts.get<by_balance>();
    for (const auto& acc : balance_idx) {
        std::cout << acc.name << ": $" << acc.balance << std::endl;
    }

    return 0;
}
```

---

## 序列索引（类似 std::list）

```cpp
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/sequenced_index.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/member.hpp>
#include <iostream>
#include <string>

using namespace boost::multi_index;

struct Task {
    int priority;
    std::string description;

    Task(int p, const std::string& d) : priority(p), description(d) {}
};

struct by_sequence {};
struct by_priority {};

typedef multi_index_container<
    Task,
    indexed_by<
        sequenced<tag<by_sequence>>,  // 保持插入顺序
        ordered_non_unique<tag<by_priority>, member<Task, int, &Task::priority>>
    >
> TaskList;

int main() {
    TaskList tasks;

    tasks.push_back(Task(2, "Write report"));
    tasks.push_back(Task(1, "Fix critical bug"));
    tasks.push_back(Task(3, "Update docs"));
    tasks.push_back(Task(1, "Review code"));

    // 按插入顺序查看
    std::cout << "Tasks in insertion order:\n";
    auto& seq_idx = tasks.get<by_sequence>();
    for (const auto& task : seq_idx) {
        std::cout << "[P" << task.priority << "] " << task.description << std::endl;
    }

    // 按优先级排序查看
    std::cout << "\nTasks by priority:\n";
    auto& priority_idx = tasks.get<by_priority>();
    for (const auto& task : priority_idx) {
        std::cout << "[P" << task.priority << "] " << task.description << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Multi-index 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/multi_index/doc/index.html)
- [Boost.Multi-index 教程](https://www.boost.org/doc/libs/1_90_0/libs/multi_index/doc/tutorial/index.html)

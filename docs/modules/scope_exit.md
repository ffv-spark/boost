# Boost.Scope_Exit - 作用域退出处理

## 概述

Boost.Scope_Exit 提供作用域退出时自动执行代码的功能，用于资源管理。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>
#include <fstream>

int main() {
    std::ofstream file("test.txt");

    BOOST_SCOPE_EXIT(&file) {
        file.close();
        std::cout << "File closed" << std::endl;
    } BOOST_SCOPE_EXIT_END

    file << "Hello, World!" << std::endl;

    // file 会在作用域结束时自动关闭

    return 0;
}
```

---

## 捕获变量

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    bool success = false;

    BOOST_SCOPE_EXIT(&vec, &success) {
        if (!success) {
            std::cout << "Cleaning up..." << std::endl;
            vec.clear();
        }
    } BOOST_SCOPE_EXIT_END

    vec.push_back(1);
    vec.push_back(2);
    vec.push_back(3);

    // 模拟失败
    if (vec.size() < 10) {
        std::cout << "Operation failed" << std::endl;
        return 1;
    }

    success = true;
    return 0;
}
```

---

## 资源管理示例

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>
#include <cstdio>

void process_file(const char* filename) {
    FILE* fp = fopen(filename, "r");

    if (!fp) {
        std::cerr << "Failed to open file" << std::endl;
        return;
    }

    BOOST_SCOPE_EXIT(fp) {
        fclose(fp);
        std::cout << "File closed" << std::endl;
    } BOOST_SCOPE_EXIT_END

    // 处理文件
    char buffer[256];
    while (fgets(buffer, sizeof(buffer), fp)) {
        std::cout << buffer;
    }

    // fp 自动关闭，即使发生异常
}

int main() {
    process_file("test.txt");
    return 0;
}
```

---

## 数据库事务

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>

class Database {
public:
    void begin_transaction() {
        std::cout << "BEGIN TRANSACTION" << std::endl;
    }

    void commit() {
        std::cout << "COMMIT" << std::endl;
    }

    void rollback() {
        std::cout << "ROLLBACK" << std::endl;
    }

    void execute(const std::string& sql) {
        std::cout << "EXECUTE: " << sql << std::endl;
    }
};

void transfer_money(Database& db, int amount) {
    db.begin_transaction();

    bool committed = false;

    BOOST_SCOPE_EXIT(&db, &committed) {
        if (!committed) {
            db.rollback();
        }
    } BOOST_SCOPE_EXIT_END

    db.execute("UPDATE accounts SET balance = balance - " +
               std::to_string(amount) + " WHERE id = 1");
    db.execute("UPDATE accounts SET balance = balance + " +
               std::to_string(amount) + " WHERE id = 2");

    // 模拟错误
    if (amount > 1000) {
        throw std::runtime_error("Amount too large");
    }

    db.commit();
    committed = true;
}

int main() {
    Database db;

    try {
        transfer_money(db, 500);   // 成功
        transfer_money(db, 2000);  // 失败，自动回滚
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## C++11 Lambda 替代

```cpp
#include <boost/scope_exit.hpp>
#include <iostream>

// BOOST_SCOPE_EXIT 方式
void method1() {
    int* ptr = new int(42);

    BOOST_SCOPE_EXIT(ptr) {
        delete ptr;
        std::cout << "Deleted (BOOST_SCOPE_EXIT)" << std::endl;
    } BOOST_SCOPE_EXIT_END

    std::cout << "Value: " << *ptr << std::endl;
}

// C++11 lambda + RAII 方式
void method2() {
    int* ptr = new int(42);

    struct Guard {
        int** p;
        ~Guard() {
            delete *p;
            std::cout << "Deleted (RAII)" << std::endl;
        }
    } guard{&ptr};

    std::cout << "Value: " << *ptr << std::endl;
}

int main() {
    method1();
    method2();
    return 0;
}
```

---

## 最佳实践

1. **异常安全**: 确保资源总是释放
2. **简单场景**: 复杂资源使用 RAII 类
3. **捕获**: 明确列出捕获的变量
4. **C++11**: 考虑使用 lambda + 智能指针
5. **性能**: 零开销抽象

---

## 参考资源

- [Boost.Scope_Exit 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/scope_exit/doc/html/index.html)

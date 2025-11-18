# Boost.Variant - 变体类型库

## 概述

Boost.Variant 提供了类型安全的联合类型，可以在编译时确定的多个类型中存储一个值。

**类型**: 仅头文件库

**注意**: C++17 已引入 `std::variant`

**主要特性**:
- 类型安全的联合
- 不会为空（总是持有某个值）
- 支持访问者模式
- 编译期类型检查

---

## 快速开始

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    // 定义可以存储 int 或 string 的变体
    boost::variant<int, std::string> v;

    v = 42;
    std::cout << "整数: " << boost::get<int>(v) << std::endl;

    v = std::string("Hello");
    std::cout << "字符串: " << boost::get<std::string>(v) << std::endl;

    // 使用访问者
    boost::apply_visitor([](const auto& value) {
        std::cout << "值: " << value << std::endl;
    }, v);

    return 0;
}
```

**编译**:
```bash
g++ -std=c++14 example.cpp -o example
```

---

## 基本用法

### 创建和赋值

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    // 1. 默认构造（使用第一个类型的默认值）
    boost::variant<int, double, std::string> v1;  // int(0)

    // 2. 直接赋值
    boost::variant<int, double, std::string> v2 = 42;
    boost::variant<int, double, std::string> v3 = 3.14;
    boost::variant<int, double, std::string> v4 = std::string("hello");

    // 3. 拷贝构造
    boost::variant<int, double> v5 = v2;

    // 4. 改变值和类型
    v2 = 100;           // 仍是 int
    v2 = 2.718;         // 现在是 double
    v2 = std::string("world");  // 现在是 string

    std::cout << "v1 类型索引: " << v1.which() << std::endl;
    std::cout << "v2 类型索引: " << v2.which() << std::endl;

    return 0;
}
```

### 访问值

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

int main() {
    boost::variant<int, double, std::string> v = 42;

    // 1. get<T>() - 精确类型
    try {
        int i = boost::get<int>(v);
        std::cout << "整数: " << i << std::endl;

        // 错误的类型会抛出异常
        double d = boost::get<double>(v);  // 抛出 bad_get
    } catch (const boost::bad_get& e) {
        std::cout << "类型不匹配" << std::endl;
    }

    // 2. get<T>() - 返回指针（安全）
    if (int* p = boost::get<int>(&v)) {
        std::cout << "整数指针: " << *p << std::endl;
    } else {
        std::cout << "不是整数" << std::endl;
    }

    // 3. which() - 获取类型索引
    std::cout << "类型索引: " << v.which() << std::endl;  // 0 for int

    // 4. type() - 获取类型信息
    std::cout << "类型: " << v.type().name() << std::endl;

    return 0;
}
```

---

## 访问者模式

### 基本访问者

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 函数对象访问者
struct PrintVisitor : public boost::static_visitor<void> {
    void operator()(int i) const {
        std::cout << "整数: " << i << std::endl;
    }

    void operator()(double d) const {
        std::cout << "浮点数: " << d << std::endl;
    }

    void operator()(const std::string& s) const {
        std::cout << "字符串: " << s << std::endl;
    }
};

int main() {
    boost::variant<int, double, std::string> v1 = 42;
    boost::variant<int, double, std::string> v2 = 3.14;
    boost::variant<int, double, std::string> v3 = std::string("hello");

    PrintVisitor visitor;

    boost::apply_visitor(visitor, v1);
    boost::apply_visitor(visitor, v2);
    boost::apply_visitor(visitor, v3);

    return 0;
}
```

### 返回值的访问者

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 转换为字符串的访问者
struct ToStringVisitor : public boost::static_visitor<std::string> {
    std::string operator()(int i) const {
        return std::to_string(i);
    }

    std::string operator()(double d) const {
        return std::to_string(d);
    }

    std::string operator()(const std::string& s) const {
        return s;
    }
};

int main() {
    boost::variant<int, double, std::string> v1 = 42;
    boost::variant<int, double, std::string> v2 = 3.14;

    ToStringVisitor to_string;

    std::string s1 = boost::apply_visitor(to_string, v1);
    std::string s2 = boost::apply_visitor(to_string, v2);

    std::cout << "s1: " << s1 << std::endl;
    std::cout << "s2: " << s2 << std::endl;

    return 0;
}
```

### Lambda 访问者（C++14）

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 辅助模板
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts> overloaded(Ts...) -> overloaded<Ts...>;

int main() {
    boost::variant<int, double, std::string> v = 42;

    // 使用 lambda
    boost::apply_visitor(
        overloaded {
            [](int i) { std::cout << "整数: " << i << std::endl; },
            [](double d) { std::cout << "浮点数: " << d << std::endl; },
            [](const std::string& s) { std::cout << "字符串: " << s << std::endl; }
        },
        v
    );

    return 0;
}
```

---

## 实用示例

### 表达式求值

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <memory>

// 前向声明
struct Expr;

using ExprPtr = std::shared_ptr<Expr>;
using Value = boost::variant<int, double>;

// 表达式节点
struct Literal { Value value; };
struct Add { ExprPtr left, right; };
struct Multiply { ExprPtr left, right; };

using ExprVariant = boost::variant<Literal, Add, Multiply>;

struct Expr {
    ExprVariant expr;

    Expr(const ExprVariant& e) : expr(e) {}
};

// 求值访问者
struct EvalVisitor : public boost::static_visitor<double> {
    double operator()(const Literal& lit) const {
        return boost::apply_visitor(*this, lit.value);
    }

    double operator()(int i) const { return i; }
    double operator()(double d) const { return d; }

    double operator()(const Add& add) const {
        return eval(add.left) + eval(add.right);
    }

    double operator()(const Multiply& mul) const {
        return eval(mul.left) * eval(mul.right);
    }

    double eval(const ExprPtr& expr) const {
        return boost::apply_visitor(*this, expr->expr);
    }
};

int main() {
    // 构建表达式: (2 + 3) * 4
    auto two = std::make_shared<Expr>(Literal{2});
    auto three = std::make_shared<Expr>(Literal{3});
    auto four = std::make_shared<Expr>(Literal{4});

    auto add = std::make_shared<Expr>(Add{two, three});
    auto expr = std::make_shared<Expr>(Multiply{add, four});

    EvalVisitor eval;
    double result = eval.eval(expr);

    std::cout << "结果: " << result << std::endl;  // 20

    return 0;
}
```

### JSON 值表示

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>
#include <map>
#include <vector>
#include <memory>

// 前向声明
struct JsonValue;

using JsonNull = boost::blank;
using JsonBool = bool;
using JsonNumber = double;
using JsonString = std::string;
using JsonArray = std::vector<std::shared_ptr<JsonValue>>;
using JsonObject = std::map<std::string, std::shared_ptr<JsonValue>>;

using JsonVariant = boost::variant<
    JsonNull,
    JsonBool,
    JsonNumber,
    JsonString,
    JsonArray,
    JsonObject
>;

struct JsonValue {
    JsonVariant value;

    JsonValue() : value(JsonNull()) {}
    JsonValue(const JsonVariant& v) : value(v) {}
};

// 打印访问者
struct PrintJsonVisitor : public boost::static_visitor<void> {
    void operator()(const JsonNull&) const {
        std::cout << "null";
    }

    void operator()(JsonBool b) const {
        std::cout << (b ? "true" : "false");
    }

    void operator()(JsonNumber n) const {
        std::cout << n;
    }

    void operator()(const JsonString& s) const {
        std::cout << "\"" << s << "\"";
    }

    void operator()(const JsonArray& arr) const {
        std::cout << "[";
        for (size_t i = 0; i < arr.size(); ++i) {
            if (i > 0) std::cout << ", ";
            boost::apply_visitor(*this, arr[i]->value);
        }
        std::cout << "]";
    }

    void operator()(const JsonObject& obj) const {
        std::cout << "{";
        bool first = true;
        for (const auto& [key, val] : obj) {
            if (!first) std::cout << ", ";
            first = false;
            std::cout << "\"" << key << "\": ";
            boost::apply_visitor(*this, val->value);
        }
        std::cout << "}";
    }
};

int main() {
    // 创建 JSON: {"name": "Alice", "age": 30, "scores": [85, 90, 95]}
    JsonObject obj;
    obj["name"] = std::make_shared<JsonValue>(JsonString("Alice"));
    obj["age"] = std::make_shared<JsonValue>(JsonNumber(30));

    JsonArray scores;
    scores.push_back(std::make_shared<JsonValue>(JsonNumber(85)));
    scores.push_back(std::make_shared<JsonValue>(JsonNumber(90)));
    scores.push_back(std::make_shared<JsonValue>(JsonNumber(95)));
    obj["scores"] = std::make_shared<JsonValue>(scores);

    JsonValue root(obj);

    PrintJsonVisitor printer;
    boost::apply_visitor(printer, root.value);
    std::cout << std::endl;

    return 0;
}
```

### 状态机

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <string>

// 状态定义
struct Idle {};
struct Running { int progress; };
struct Paused { int saved_progress; };
struct Completed {};

using State = boost::variant<Idle, Running, Paused, Completed>;

// 事件定义
struct Start {};
struct Pause {};
struct Resume {};
struct Finish {};

// 状态转换访问者
struct TransitionVisitor : public boost::static_visitor<State> {
    // Idle + Start -> Running
    State operator()(const Idle&, const Start&) const {
        std::cout << "启动" << std::endl;
        return Running{0};
    }

    // Running + Pause -> Paused
    State operator()(const Running& r, const Pause&) const {
        std::cout << "暂停在进度 " << r.progress << std::endl;
        return Paused{r.progress};
    }

    // Paused + Resume -> Running
    State operator()(const Paused& p, const Resume&) const {
        std::cout << "恢复从进度 " << p.saved_progress << std::endl;
        return Running{p.saved_progress};
    }

    // Running + Finish -> Completed
    State operator()(const Running&, const Finish&) const {
        std::cout << "完成" << std::endl;
        return Completed{};
    }

    // 默认：无效转换
    template<typename S, typename E>
    State operator()(const S&, const E&) const {
        std::cout << "无效转换" << std::endl;
        return S{};
    }
};

class StateMachine {
public:
    StateMachine() : state_(Idle{}) {}

    template<typename Event>
    void process(const Event& event) {
        state_ = boost::apply_visitor(
            [&event](const auto& state) {
                return TransitionVisitor()(state, event);
            },
            state_
        );
    }

    void print_state() const {
        boost::apply_visitor(
            overloaded {
                [](const Idle&) { std::cout << "状态: 空闲" << std::endl; },
                [](const Running& r) { std::cout << "状态: 运行中 (" << r.progress << "%)" << std::endl; },
                [](const Paused& p) { std::cout << "状态: 已暂停 (" << p.saved_progress << "%)" << std::endl; },
                [](const Completed&) { std::cout << "状态: 已完成" << std::endl; }
            },
            state_
        );
    }

private:
    State state_;

    template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
};

int main() {
    StateMachine sm;

    sm.print_state();
    sm.process(Start{});
    sm.print_state();

    sm.process(Pause{});
    sm.print_state();

    sm.process(Resume{});
    sm.print_state();

    sm.process(Finish{});
    sm.print_state();

    return 0;
}
```

---

## 递归变体

```cpp
#include <boost/variant.hpp>
#include <iostream>
#include <vector>
#include <memory>

// 前向声明
struct TreeNode;

// 使用 recursive_wrapper 支持递归
using Tree = boost::variant<
    int,  // 叶子节点
    boost::recursive_wrapper<std::vector<Tree>>  // 分支节点
>;

struct TreeNode {
    Tree tree;

    TreeNode(int value) : tree(value) {}
    TreeNode(const std::vector<Tree>& children) : tree(children) {}
};

// 打印树的访问者
struct PrintTreeVisitor : public boost::static_visitor<void> {
    PrintTreeVisitor(int indent = 0) : indent_(indent) {}

    void operator()(int value) const {
        std::cout << std::string(indent_, ' ') << value << std::endl;
    }

    void operator()(const std::vector<Tree>& children) const {
        std::cout << std::string(indent_, ' ') << "[" << std::endl;
        for (const auto& child : children) {
            boost::apply_visitor(PrintTreeVisitor(indent_ + 2), child);
        }
        std::cout << std::string(indent_, ' ') << "]" << std::endl;
    }

private:
    int indent_;
};

int main() {
    // 构建树: [1, [2, 3], 4]
    Tree tree = std::vector<Tree>{
        Tree(1),
        Tree(std::vector<Tree>{Tree(2), Tree(3)}),
        Tree(4)
    };

    boost::apply_visitor(PrintTreeVisitor(), tree);

    return 0;
}
```

---

## 最佳实践

1. **使用访问者**: 避免大量的 if-else 或 switch
2. **避免空状态**: Variant 总是有值
3. **异常安全**: 使用指针版本的 get
4. **类型索引**: 用 which() 高效检查类型
5. **C++17+**: 优先使用 std::variant

---

## 参考资源

- [Boost.Variant 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/variant.html)
- [Boost.Variant 教程](https://www.boost.org/doc/libs/1_90_0/doc/html/variant/tutorial.html)

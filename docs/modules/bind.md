# Boost.Bind 和 Boost.Function - 函数对象库

## 概述

Boost.Bind 提供函数参数绑定，Boost.Function 提供函数对象封装。

**类型**: 仅头文件库

**注意**: C++11 已引入 `std::bind` 和 `std::function`

---

## Boost.Function

### 基本用法

```cpp
#include <boost/function.hpp>
#include <iostream>

int add(int a, int b) {
    return a + b;
}

struct Multiplier {
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    // 函数指针
    boost::function<int(int, int)> f1 = add;
    std::cout << "add(2, 3) = " << f1(2, 3) << std::endl;

    // 函数对象
    boost::function<int(int, int)> f2 = Multiplier();
    std::cout << "multiply(2, 3) = " << f2(2, 3) << std::endl;

    // Lambda
    boost::function<int(int, int)> f3 = [](int a, int b) { return a - b; };
    std::cout << "subtract(5, 3) = " << f3(5, 3) << std::endl;

    // 检查是否为空
    boost::function<void()> empty;
    if (!empty) {
        std::cout << "Function is empty" << std::endl;
    }

    return 0;
}
```

### 回调函数

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <vector>

class Button {
public:
    using ClickHandler = boost::function<void()>;

    void set_click_handler(ClickHandler handler) {
        click_handler_ = handler;
    }

    void click() {
        if (click_handler_) {
            click_handler_();
        }
    }

private:
    ClickHandler click_handler_;
};

int main() {
    Button button;

    button.set_click_handler([]() {
        std::cout << "Button clicked!" << std::endl;
    });

    button.click();

    return 0;
}
```

---

## Boost.Bind

### 基本绑定

```cpp
#include <boost/bind/bind.hpp>
#include <boost/function.hpp>
#include <iostream>

using namespace boost::placeholders;

int add(int a, int b) {
    return a + b;
}

int main() {
    // 绑定第一个参数
    auto add5 = boost::bind(add, 5, _1);
    std::cout << "5 + 3 = " << add5(3) << std::endl;

    // 绑定第二个参数
    auto add10 = boost::bind(add, _1, 10);
    std::cout << "7 + 10 = " << add10(7) << std::endl;

    // 绑定两个参数
    auto add_5_10 = boost::bind(add, 5, 10);
    std::cout << "5 + 10 = " << add_5_10() << std::endl;

    // 参数重排序
    auto reversed_add = boost::bind(add, _2, _1);
    std::cout << "reversed: " << reversed_add(3, 5) << std::endl;

    return 0;
}
```

### 成员函数绑定

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <string>

using namespace boost::placeholders;

class Person {
public:
    Person(const std::string& name) : name_(name) {}

    void greet() const {
        std::cout << "Hello, I'm " << name_ << std::endl;
    }

    void set_name(const std::string& name) {
        name_ = name;
    }

    std::string get_name() const {
        return name_;
    }

private:
    std::string name_;
};

int main() {
    Person alice("Alice");
    Person bob("Bob");

    // 绑定成员函数
    auto greet = boost::bind(&Person::greet, _1);
    greet(alice);
    greet(bob);

    // 绑定对象和成员函数
    auto alice_greet = boost::bind(&Person::greet, &alice);
    alice_greet();

    // 绑定带参数的成员函数
    auto set_name = boost::bind(&Person::set_name, _1, _2);
    set_name(&alice, "Alice2");
    alice.greet();

    return 0;
}
```

### STL 算法中使用

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

bool greater_than(int a, int threshold) {
    return a > threshold;
}

int main() {
    std::vector<int> nums = {1, 5, 3, 8, 2, 9, 4};

    // 查找大于5的元素
    auto it = std::find_if(nums.begin(), nums.end(),
                           boost::bind(greater_than, _1, 5));

    if (it != nums.end()) {
        std::cout << "Found: " << *it << std::endl;
    }

    // 统计大于5的元素
    int count = std::count_if(nums.begin(), nums.end(),
                              boost::bind(greater_than, _1, 5));
    std::cout << "Count: " << count << std::endl;

    return 0;
}
```

---

## 实用示例

### 事件系统

```cpp
#include <boost/function.hpp>
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <string>

using namespace boost::placeholders;

class EventManager {
public:
    using EventHandler = boost::function<void(const std::string&)>;

    void subscribe(const std::string& event, EventHandler handler) {
        handlers_[event].push_back(handler);
    }

    void emit(const std::string& event, const std::string& data) {
        auto it = handlers_.find(event);
        if (it != handlers_.end()) {
            for (auto& handler : it->second) {
                handler(data);
            }
        }
    }

private:
    std::map<std::string, std::vector<EventHandler>> handlers_;
};

class Logger {
public:
    void on_event(const std::string& data) {
        std::cout << "[Logger] Event: " << data << std::endl;
    }
};

int main() {
    EventManager events;
    Logger logger;

    // 订阅事件
    events.subscribe("user_login",
        boost::bind(&Logger::on_event, &logger, _1));

    events.subscribe("user_login", [](const std::string& data) {
        std::cout << "[Handler] User logged in: " << data << std::endl;
    });

    // 触发事件
    events.emit("user_login", "Alice");

    return 0;
}
```

### 延迟执行

```cpp
#include <boost/function.hpp>
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>

using namespace boost::placeholders;

class TaskQueue {
public:
    using Task = boost::function<void()>;

    void add_task(Task task) {
        tasks_.push_back(task);
    }

    void execute_all() {
        for (auto& task : tasks_) {
            task();
        }
        tasks_.clear();
    }

private:
    std::vector<Task> tasks_;
};

void print_sum(int a, int b) {
    std::cout << a << " + " << b << " = " << (a + b) << std::endl;
}

int main() {
    TaskQueue queue;

    // 添加延迟任务
    queue.add_task(boost::bind(print_sum, 2, 3));
    queue.add_task(boost::bind(print_sum, 5, 7));
    queue.add_task(boost::bind(print_sum, 10, 20));

    std::cout << "Executing tasks..." << std::endl;
    queue.execute_all();

    return 0;
}
```

---

## 与 std::function/std::bind 对比

```cpp
#include <boost/function.hpp>
#include <boost/bind/bind.hpp>
#include <functional>
#include <iostream>

int add(int a, int b) { return a + b; }

int main() {
    // Boost
    boost::function<int(int, int)> b_func = add;
    auto b_bind = boost::bind(add, 5, boost::placeholders::_1);

    // Standard (C++11)
    std::function<int(int, int)> s_func = add;
    auto s_bind = std::bind(add, 5, std::placeholders::_1);

    std::cout << "Boost: " << b_bind(3) << std::endl;
    std::cout << "Std: " << s_bind(3) << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **C++11+**: 优先使用 std::function 和 std::bind
2. **Lambda**: 简单情况使用 lambda 更清晰
3. **性能**: function 有轻微开销
4. **类型擦除**: function 可存储任意可调用对象
5. **空检查**: 调用前检查 function 是否为空

---

## 参考资源

- [Boost.Function 文档](https://www.boost.org/doc/libs/1_90_0/doc/html/function.html)
- [Boost.Bind 文档](https://www.boost.org/doc/libs/1_90_0/libs/bind/doc/html/bind.html)

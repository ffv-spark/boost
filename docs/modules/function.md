# Boost.Function - 函数对象包装库

## 概述

Boost.Function 提供类型安全的函数对象包装器，可以存储任何可调用对象。

**类型**: 仅头文件库

**注意**: C++11 引入了 `std::function`，建议使用标准库版本

---

## 快速开始

```cpp
#include <boost/function.hpp>
#include <iostream>

void print_message(const std::string& msg) {
    std::cout << msg << std::endl;
}

int main() {
    // 包装普通函数
    boost::function<void(const std::string&)> func = print_message;

    func("Hello, Boost.Function!");

    return 0;
}
```

---

## 包装不同类型的可调用对象

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <functional>

// 普通函数
int add(int a, int b) {
    return a + b;
}

// 函数对象
struct Multiplier {
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    typedef boost::function<int(int, int)> BinaryOp;

    // 包装普通函数
    BinaryOp op1 = add;
    std::cout << "加法: " << op1(3, 4) << std::endl;

    // 包装函数对象
    BinaryOp op2 = Multiplier();
    std::cout << "乘法: " << op2(3, 4) << std::endl;

    // 包装 lambda 表达式
    BinaryOp op3 = [](int a, int b) { return a - b; };
    std::cout << "减法: " << op3(10, 3) << std::endl;

    // 包装 std::bind
    BinaryOp op4 = std::bind(std::divides<int>(), std::placeholders::_1, std::placeholders::_2);
    std::cout << "除法: " << op4(12, 3) << std::endl;

    return 0;
}
```

---

## 检查函数是否为空

```cpp
#include <boost/function.hpp>
#include <iostream>

int square(int x) {
    return x * x;
}

int main() {
    boost::function<int(int)> func;

    // 检查是否为空
    if (!func) {
        std::cout << "函数为空" << std::endl;
    }

    func = square;

    if (func) {
        std::cout << "函数不为空，调用结果: " << func(5) << std::endl;
    }

    // 清空函数
    func = nullptr;

    if (!func) {
        std::cout << "函数已清空" << std::endl;
    }

    return 0;
}
```

---

## 成员函数包装

```cpp
#include <boost/function.hpp>
#include <boost/bind.hpp>
#include <iostream>
#include <string>

class Person {
public:
    Person(const std::string& name) : name_(name) {}

    void greet(const std::string& greeting) const {
        std::cout << greeting << ", 我是 " << name_ << std::endl;
    }

    int add(int a, int b) const {
        return a + b;
    }

private:
    std::string name_;
};

int main() {
    Person alice("Alice");

    // 包装成员函数
    boost::function<void(const std::string&)> greet_func =
        boost::bind(&Person::greet, &alice, _1);

    greet_func("你好");

    // 包装带返回值的成员函数
    boost::function<int(int, int)> add_func =
        boost::bind(&Person::add, &alice, _1, _2);

    std::cout << "结果: " << add_func(3, 4) << std::endl;

    return 0;
}
```

---

## 回调机制

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <vector>

class Button {
public:
    typedef boost::function<void()> ClickCallback;

    void set_click_handler(ClickCallback callback) {
        click_callback_ = callback;
    }

    void click() {
        if (click_callback_) {
            std::cout << "按钮被点击" << std::endl;
            click_callback_();
        }
    }

private:
    ClickCallback click_callback_;
};

void on_button_click() {
    std::cout << "处理按钮点击事件" << std::endl;
}

class Application {
public:
    void on_button_clicked() {
        std::cout << "应用程序处理点击" << std::endl;
    }
};

int main() {
    Button button;

    // 设置普通函数作为回调
    button.set_click_handler(on_button_click);
    button.click();

    std::cout << std::endl;

    // 设置 lambda 作为回调
    button.set_click_handler([]() {
        std::cout << "Lambda 处理点击" << std::endl;
    });
    button.click();

    std::cout << std::endl;

    // 设置成员函数作为回调
    Application app;
    button.set_click_handler(boost::bind(&Application::on_button_clicked, &app));
    button.click();

    return 0;
}
```

---

## 多个回调（观察者模式）

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <vector>
#include <string>

class EventSource {
public:
    typedef boost::function<void(const std::string&)> EventCallback;

    void add_listener(EventCallback callback) {
        listeners_.push_back(callback);
    }

    void trigger_event(const std::string& data) {
        std::cout << "触发事件: " << data << std::endl;
        for (auto& listener : listeners_) {
            listener(data);
        }
    }

private:
    std::vector<EventCallback> listeners_;
};

void listener1(const std::string& data) {
    std::cout << "监听器1收到: " << data << std::endl;
}

void listener2(const std::string& data) {
    std::cout << "监听器2收到: " << data << std::endl;
}

int main() {
    EventSource source;

    source.add_listener(listener1);
    source.add_listener(listener2);
    source.add_listener([](const std::string& data) {
        std::cout << "Lambda监听器收到: " << data << std::endl;
    });

    source.trigger_event("测试事件");

    return 0;
}
```

---

## 延迟调用

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <vector>

class CommandQueue {
public:
    typedef boost::function<void()> Command;

    void enqueue(Command cmd) {
        commands_.push_back(cmd);
    }

    void execute_all() {
        for (auto& cmd : commands_) {
            cmd();
        }
        commands_.clear();
    }

private:
    std::vector<Command> commands_;
};

int main() {
    CommandQueue queue;

    int counter = 0;

    // 添加多个命令
    queue.enqueue([&counter]() {
        std::cout << "命令1: counter = " << ++counter << std::endl;
    });

    queue.enqueue([&counter]() {
        std::cout << "命令2: counter = " << counter * 2 << std::endl;
    });

    queue.enqueue([]() {
        std::cout << "命令3: Hello!" << std::endl;
    });

    std::cout << "执行所有命令:" << std::endl;
    queue.execute_all();

    return 0;
}
```

---

## 策略模式

```cpp
#include <boost/function.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

class Sorter {
public:
    typedef boost::function<bool(int, int)> CompareFunc;

    void set_compare_function(CompareFunc func) {
        compare_ = func;
    }

    void sort(std::vector<int>& data) {
        if (compare_) {
            std::sort(data.begin(), data.end(), compare_);
        }
    }

private:
    CompareFunc compare_;
};

int main() {
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3};

    Sorter sorter;

    // 升序排序
    sorter.set_compare_function([](int a, int b) { return a < b; });
    sorter.sort(numbers);

    std::cout << "升序: ";
    for (int n : numbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    // 降序排序
    sorter.set_compare_function([](int a, int b) { return a > b; });
    sorter.sort(numbers);

    std::cout << "降序: ";
    for (int n : numbers) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 与 std::function 对比

```cpp
#include <boost/function.hpp>
#include <functional>
#include <iostream>

int multiply(int a, int b) {
    return a * b;
}

int main() {
    // Boost.Function
    boost::function<int(int, int)> boost_func = multiply;
    std::cout << "Boost.Function: " << boost_func(3, 4) << std::endl;

    // std::function (C++11)
    std::function<int(int, int)> std_func = multiply;
    std::cout << "std::function: " << std_func(3, 4) << std::endl;

    // 两者用法基本相同
    boost_func = [](int a, int b) { return a + b; };
    std_func = [](int a, int b) { return a + b; };

    std::cout << "Lambda - Boost: " << boost_func(5, 6) << std::endl;
    std::cout << "Lambda - std: " << std_func(5, 6) << std::endl;

    return 0;
}
```

**建议**: 在现代 C++ 项目中，优先使用 `std::function`。

---

## 参考资源

- [Boost.Function 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/function.html)
- [std::function 文档](https://en.cppreference.com/w/cpp/utility/functional/function)

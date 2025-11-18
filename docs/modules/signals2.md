# Boost.Signals2 - 信号槽库

## 概述

Boost.Signals2 提供线程安全的信号槽机制，实现观察者模式。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/signals2.hpp>
#include <iostream>

void hello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    // 创建信号
    boost::signals2::signal<void()> sig;

    // 连接槽
    sig.connect(&hello);

    // 触发信号
    sig();

    return 0;
}
```

---

## 基本信号

```cpp
#include <boost/signals2.hpp>
#include <iostream>

void print_number(int n) {
    std::cout << "数字: " << n << std::endl;
}

void print_square(int n) {
    std::cout << "平方: " << n * n << std::endl;
}

int main() {
    boost::signals2::signal<void(int)> sig;

    // 连接多个槽
    sig.connect(&print_number);
    sig.connect(&print_square);

    // 触发信号
    std::cout << "触发信号(5):\n";
    sig(5);

    std::cout << "\n触发信号(10):\n";
    sig(10);

    return 0;
}
```

---

## Lambda 函数

```cpp
#include <boost/signals2.hpp>
#include <iostream>

int main() {
    boost::signals2::signal<void(std::string)> sig;

    // 连接 lambda
    sig.connect([](const std::string& msg) {
        std::cout << "Lambda 1: " << msg << std::endl;
    });

    sig.connect([](const std::string& msg) {
        std::cout << "Lambda 2: " << msg << " (长度: " << msg.length() << ")" << std::endl;
    });

    sig("Hello, Signals!");

    return 0;
}
```

---

## 连接管理

```cpp
#include <boost/signals2.hpp>
#include <iostream>

void slot1() {
    std::cout << "槽1被调用" << std::endl;
}

void slot2() {
    std::cout << "槽2被调用" << std::endl;
}

int main() {
    boost::signals2::signal<void()> sig;

    // 保存连接
    auto conn1 = sig.connect(&slot1);
    auto conn2 = sig.connect(&slot2);

    std::cout << "第一次触发:\n";
    sig();

    // 断开连接
    conn1.disconnect();

    std::cout << "\n断开槽1后:\n";
    sig();

    return 0;
}
```

---

## 返回值合并

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <vector>

int compute1(int x) {
    return x * 2;
}

int compute2(int x) {
    return x + 10;
}

int compute3(int x) {
    return x * x;
}

int main() {
    // 默认返回最后一个槽的值
    boost::signals2::signal<int(int)> sig;

    sig.connect(&compute1);
    sig.connect(&compute2);
    sig.connect(&compute3);

    int result = sig(5);
    std::cout << "最后返回值: " << result << std::endl;  // 25 (5*5)

    return 0;
}
```

---

## 自定义合并器

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <vector>
#include <numeric>

// 求和合并器
template<typename T>
struct sum_combiner {
    typedef T result_type;

    template<typename InputIterator>
    T operator()(InputIterator first, InputIterator last) const {
        if (first == last) return T();
        return std::accumulate(first, last, T());
    }
};

int add_ten(int x) { return x + 10; }
int multiply_two(int x) { return x * 2; }
int square(int x) { return x * x; }

int main() {
    // 使用求和合并器
    boost::signals2::signal<int(int), sum_combiner<int>> sig;

    sig.connect(&add_ten);
    sig.connect(&multiply_two);
    sig.connect(&square);

    int result = sig(5);
    std::cout << "所有返回值之和: " << result << std::endl;  // 15 + 10 + 25 = 50

    return 0;
}
```

---

## 槽分组

```cpp
#include <boost/signals2.hpp>
#include <iostream>

void high_priority() {
    std::cout << "高优先级" << std::endl;
}

void normal_priority() {
    std::cout << "普通优先级" << std::endl;
}

void low_priority() {
    std::cout << "低优先级" << std::endl;
}

int main() {
    boost::signals2::signal<void()> sig;

    // 按组连接
    sig.connect(2, &low_priority);     // 组2
    sig.connect(0, &high_priority);    // 组0
    sig.connect(1, &normal_priority);  // 组1

    std::cout << "触发信号（按组顺序）:\n";
    sig();

    return 0;
}
```

---

## 自动断开

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <memory>

class Button {
public:
    boost::signals2::signal<void()> onClick;
};

class Window {
public:
    void handleClick() {
        std::cout << "窗口处理点击" << std::endl;
    }
};

int main() {
    Button button;
    auto window = std::make_shared<Window>();

    // 使用 weak_ptr 跟踪对象
    button.onClick.connect(
        boost::signals2::signal<void()>::slot_type(
            &Window::handleClick, window.get()
        ).track(window)
    );

    std::cout << "窗口存在时点击:\n";
    button.onClick();

    // 释放窗口
    window.reset();

    std::cout << "\n窗口释放后点击:\n";
    button.onClick();  // 不会调用槽

    return 0;
}
```

---

## 线程安全

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::signals2::signal<void(int)> sig;

    sig.connect([](int id) {
        std::cout << "线程 " << id << " 收到信号" << std::endl;
    });

    // 从多个线程触发信号（Signals2 是线程安全的）
    std::thread t1([&sig]() {
        for (int i = 0; i < 3; ++i) {
            sig(1);
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });

    std::thread t2([&sig]() {
        for (int i = 0; i < 3; ++i) {
            sig(2);
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });

    t1.join();
    t2.join();

    return 0;
}
```

---

## GUI 事件示例

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <string>

class Button {
public:
    boost::signals2::signal<void()> clicked;
    boost::signals2::signal<void()> hovered;

    void click() {
        std::cout << "[按钮被点击]" << std::endl;
        clicked();
    }

    void hover() {
        std::cout << "[鼠标悬停]" << std::endl;
        hovered();
    }
};

class Application {
public:
    void onButtonClicked() {
        std::cout << "  应用程序处理点击" << std::endl;
    }

    void onButtonHovered() {
        std::cout << "  应用程序处理悬停" << std::endl;
    }
};

int main() {
    Button button;
    Application app;

    // 连接事件处理器
    button.clicked.connect(
        boost::bind(&Application::onButtonClicked, &app)
    );
    button.hovered.connect(
        boost::bind(&Application::onButtonHovered, &app)
    );

    // 模拟事件
    button.click();
    button.hover();
    button.click();

    return 0;
}
```

---

## 参考资源

- [Boost.Signals2 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/signals2.html)

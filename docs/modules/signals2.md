# Boost.Signals2 - 信号/槽机制

## 概述

Boost.Signals2 实现了线程安全的信号/槽机制，用于对象间解耦通信。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/signals2.hpp>
#include <iostream>

void hello() {
    std::cout << "Hello, ";
}

void world() {
    std::cout << "World!" << std::endl;
}

int main() {
    boost::signals2::signal<void()> sig;

    // 连接槽函数
    sig.connect(&hello);
    sig.connect(&world);

    // 触发信号
    sig();

    return 0;
}
```

---

## 带参数的信号

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <string>

int main() {
    boost::signals2::signal<void(const std::string&, int)> sig;

    // 连接槽
    sig.connect([](const std::string& msg, int value) {
        std::cout << msg << ": " << value << std::endl;
    });

    sig.connect([](const std::string& msg, int value) {
        std::cout << "Double: " << (value * 2) << std::endl;
    });

    // 触发
    sig("Number", 42);

    return 0;
}
```

---

## 返回值处理

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <vector>

int double_value(int x) { return x * 2; }
int triple_value(int x) { return x * 3; }

int main() {
    boost::signals2::signal<int(int)> sig;

    sig.connect(&double_value);
    sig.connect(&triple_value);

    // 默认返回最后一个槽的结果
    auto result = sig(10);
    if (result) {
        std::cout << "Last result: " << *result << std::endl;
    }

    // 自定义合并器：收集所有结果
    boost::signals2::signal<int(int),
        boost::signals2::optional_last_value<int>,
        int,
        std::less<int>,
        boost::function<int(int)>,
        boost::function<void(const boost::signals2::connection&)>,
        boost::signals2::mutex> sig2;

    sig2.connect(&double_value);
    sig2.connect(&triple_value);

    return 0;
}
```

---

## 连接管理

```cpp
#include <boost/signals2.hpp>
#include <iostream>

int main() {
    boost::signals2::signal<void(int)> sig;

    // 连接并保存连接对象
    boost::signals2::connection conn1 = sig.connect([](int x) {
        std::cout << "Slot 1: " << x << std::endl;
    });

    auto conn2 = sig.connect([](int x) {
        std::cout << "Slot 2: " << x << std::endl;
    });

    sig(1);  // 两个槽都会被调用

    // 断开连接
    conn1.disconnect();

    sig(2);  // 只有 Slot 2 被调用

    // 阻塞连接
    {
        boost::signals2::shared_connection_block blocker(conn2);
        sig(3);  // 没有输出
    }

    sig(4);  // Slot 2 恢复

    return 0;
}
```

---

## 实用示例

### 观察者模式

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <string>

class Subject {
public:
    using Observer = boost::signals2::signal<void(const std::string&)>;

    boost::signals2::connection subscribe(const Observer::slot_type& slot) {
        return on_change_.connect(slot);
    }

    void set_data(const std::string& data) {
        data_ = data;
        on_change_(data_);
    }

private:
    std::string data_;
    Observer on_change_;
};

class Observer1 {
public:
    void update(const std::string& data) {
        std::cout << "Observer1 收到: " << data << std::endl;
    }
};

class Observer2 {
public:
    void update(const std::string& data) {
        std::cout << "Observer2 收到: " << data << std::endl;
    }
};

int main() {
    Subject subject;
    Observer1 obs1;
    Observer2 obs2;

    subject.subscribe(boost::bind(&Observer1::update, &obs1, _1));
    subject.subscribe(boost::bind(&Observer2::update, &obs2, _1));

    subject.set_data("Hello");
    subject.set_data("World");

    return 0;
}
```

### GUI 按钮

```cpp
#include <boost/signals2.hpp>
#include <iostream>

class Button {
public:
    boost::signals2::signal<void()> clicked;

    void click() {
        std::cout << "Button clicked!" << std::endl;
        clicked();
    }
};

int main() {
    Button button;

    // 连接多个处理器
    button.clicked.connect([]() {
        std::cout << "Handler 1" << std::endl;
    });

    button.clicked.connect([]() {
        std::cout << "Handler 2" << std::endl;
    });

    button.click();

    return 0;
}
```

---

## 自动断开连接

```cpp
#include <boost/signals2.hpp>
#include <iostream>
#include <memory>

class Listener {
public:
    Listener(int id) : id_(id) {
        std::cout << "Listener " << id_ << " created" << std::endl;
    }

    ~Listener() {
        std::cout << "Listener " << id_ << " destroyed" << std::endl;
    }

    void on_event() {
        std::cout << "Listener " << id_ << " handling event" << std::endl;
    }

private:
    int id_;
};

int main() {
    boost::signals2::signal<void()> sig;

    auto listener1 = std::make_shared<Listener>(1);
    auto listener2 = std::make_shared<Listener>(2);

    // 使用 track 自动管理生命周期
    sig.connect(boost::signals2::signal<void()>::slot_type(
        &Listener::on_event, listener1.get()
    ).track(listener1));

    sig.connect(boost::signals2::signal<void()>::slot_type(
        &Listener::on_event, listener2.get()
    ).track(listener2));

    std::cout << "\n触发事件 1:" << std::endl;
    sig();

    // 销毁 listener1
    listener1.reset();

    std::cout << "\n触发事件 2:" << std::endl;
    sig();  // 只有 listener2 响应

    return 0;
}
```

---

## 最佳实践

1. **线程安全**: Signals2 是线程安全的
2. **连接管理**: 保存重要的连接对象
3. **自动断开**: 使用 track 管理生命周期
4. **性能**: 避免过多的信号连接
5. **替代方案**: 考虑使用观察者模式或回调

---

## 参考资源

- [Boost.Signals2 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/signals2.html)

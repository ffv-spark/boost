# Boost.Bind - 函数绑定库

## 概述

Boost.Bind 提供函数参数绑定功能，是 C++11 std::bind 的前身。

**类型**: 仅头文件库

**注意**: C++11 引入了 std::bind 和 lambda，优先使用现代方法

---

## 快速开始

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

void print(int x) {
    std::cout << x << " ";
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 绑定函数
    std::for_each(numbers.begin(), numbers.end(), 
                  boost::bind(&print, _1));
    std::cout << std::endl;

    return 0;
}
```

---

## 绑定普通函数

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>

using namespace boost::placeholders;

int add(int a, int b) {
    return a + b;
}

int main() {
    // 绑定第一个参数为10
    auto add10 = boost::bind(add, 10, _1);

    std::cout << "10 + 5 = " << add10(5) << std::endl;
    std::cout << "10 + 20 = " << add10(20) << std::endl;

    // 绑定两个参数
    auto add_5_3 = boost::bind(add, 5, 3);
    std::cout << "5 + 3 = " << add_5_3() << std::endl;

    return 0;
}
```

---

## 占位符

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>

using namespace boost::placeholders;

int subtract(int a, int b) {
    return a - b;
}

int main() {
    // _1 表示第一个参数
    auto f1 = boost::bind(subtract, _1, 10);
    std::cout << "20 - 10 = " << f1(20) << std::endl;

    // _2 表示第二个参数
    auto f2 = boost::bind(subtract, 10, _1);
    std::cout << "10 - 5 = " << f2(5) << std::endl;

    // 交换参数顺序
    auto swap_subtract = boost::bind(subtract, _2, _1);
    std::cout << "10 - 20 = " << swap_subtract(20, 10) << std::endl;

    return 0;
}
```

---

## 绑定成员函数

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

    void set_age(int age) {
        age_ = age;
    }

    int get_age() const {
        return age_;
    }

private:
    std::string name_;
    int age_ = 0;
};

int main() {
    Person alice("Alice");
    Person bob("Bob");

    // 绑定成员函数
    auto greet_alice = boost::bind(&Person::greet, &alice);
    greet_alice();

    auto greet_bob = boost::bind(&Person::greet, &bob);
    greet_bob();

    // 绑定带参数的成员函数
    auto set_alice_age = boost::bind(&Person::set_age, &alice, _1);
    set_alice_age(30);

    auto get_alice_age = boost::bind(&Person::get_age, &alice);
    std::cout << "Alice's age: " << get_alice_age() << std::endl;

    return 0;
}
```

---

## 容器操作

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

bool greater_than(int x, int threshold) {
    return x > threshold;
}

int main() {
    std::vector<int> numbers = {1, 5, 3, 8, 2, 9, 4, 7, 6};

    // 计数大于5的元素
    int count = std::count_if(numbers.begin(), numbers.end(),
                              boost::bind(greater_than, _1, 5));
    std::cout << "大于5的元素数: " << count << std::endl;

    // 查找第一个大于6的元素
    auto it = std::find_if(numbers.begin(), numbers.end(),
                           boost::bind(greater_than, _1, 6));
    if (it != numbers.end()) {
        std::cout << "第一个大于6的元素: " << *it << std::endl;
    }

    return 0;
}
```

---

## 逻辑运算

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

bool is_even(int x) {
    return x % 2 == 0;
}

bool is_positive(int x) {
    return x > 0;
}

int main() {
    std::vector<int> numbers = {-2, -1, 0, 1, 2, 3, 4, 5};

    // 偶数且正数
    auto count = std::count_if(numbers.begin(), numbers.end(),
        boost::bind(std::logical_and<bool>(),
            boost::bind(is_even, _1),
            boost::bind(is_positive, _1)));

    std::cout << "偶数且正数的元素数: " << count << std::endl;

    return 0;
}
```

---

## 比较操作

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

int main() {
    std::vector<int> numbers = {5, 2, 8, 1, 9, 3, 7, 4, 6};

    // 降序排序
    std::sort(numbers.begin(), numbers.end(),
              boost::bind(std::greater<int>(), _1, _2));

    std::cout << "降序: ";
    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 嵌套绑定

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>

using namespace boost::placeholders;

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

int main() {
    // (x + 5) * 2
    auto f = boost::bind(multiply,
                        boost::bind(add, _1, 5),
                        2);

    std::cout << "(3 + 5) * 2 = " << f(3) << std::endl;
    std::cout << "(10 + 5) * 2 = " << f(10) << std::endl;

    return 0;
}
```

---

## 回调函数

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <string>
#include <vector>

using namespace boost::placeholders;

class Button {
public:
    typedef boost::function<void()> ClickHandler;

    void set_click_handler(ClickHandler handler) {
        handler_ = handler;
    }

    void click() {
        if (handler_) {
            handler_();
        }
    }

private:
    ClickHandler handler_;
};

class Application {
public:
    void on_button_clicked(const std::string& button_name) {
        std::cout << button_name << " 被点击" << std::endl;
    }
};

int main() {
    Application app;
    Button ok_button, cancel_button;

    // 绑定回调
    ok_button.set_click_handler(
        boost::bind(&Application::on_button_clicked, &app, "确定按钮"));

    cancel_button.set_click_handler(
        boost::bind(&Application::on_button_clicked, &app, "取消按钮"));

    // 模拟点击
    ok_button.click();
    cancel_button.click();

    return 0;
}
```

---

## 与 lambda 对比

```cpp
#include <boost/bind/bind.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

using namespace boost::placeholders;

int add(int a, int b) {
    return a + b;
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    // 使用 bind
    auto add10_bind = boost::bind(add, _1, 10);
    std::cout << "bind: " << add10_bind(5) << std::endl;

    // 使用 lambda
    auto add10_lambda = [](int x) { return add(x, 10); };
    std::cout << "lambda: " << add10_lambda(5) << std::endl;

    // bind 在某些情况下更简洁
    std::transform(numbers.begin(), numbers.end(), numbers.begin(),
                   boost::bind(add, _1, 10));

    // lambda 更灵活
    std::transform(numbers.begin(), numbers.end(), numbers.begin(),
                   [](int x) { return x + 10; });

    return 0;
}
```

---

## 参考资源

- [Boost.Bind 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/bind/doc/html/bind.html)
- [C++11 std::bind](https://en.cppreference.com/w/cpp/utility/functional/bind)

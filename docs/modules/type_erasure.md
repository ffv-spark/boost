# Boost.TypeErasure - 类型擦除库

## 概述

Boost.TypeErasure 提供运行时多态，允许在不使用继承的情况下实现类似虚函数的行为。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/operators.hpp>
#include <iostream>

namespace te = boost::type_erasure;

int main() {
    // 定义要求：可复制、可输出
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            te::typeid_<>,
            te::ostreamable<>
        >
    > printable;

    // 可以存储任何满足要求的类型
    printable x = 42;
    std::cout << x << std::endl;  // 42

    x = std::string("Hello");
    std::cout << x << std::endl;  // Hello

    x = 3.14;
    std::cout << x << std::endl;  // 3.14

    return 0;
}
```

---

## 自定义概念

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/callable.hpp>
#include <boost/type_erasure/member.hpp>
#include <iostream>
#include <string>

namespace te = boost::type_erasure;

// 定义"可说话"的概念
BOOST_TYPE_ERASURE_MEMBER((has_speak), speak, 0)

// 测试类
class Dog {
public:
    std::string speak() const { return "Woof!"; }
};

class Cat {
public:
    std::string speak() const { return "Meow!"; }
};

int main() {
    // 定义满足 has_speak 概念的类型
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            has_speak<std::string(), const te::_self>
        >
    > speakable;

    speakable animal1 = Dog();
    std::cout << animal1.speak() << std::endl;  // Woof!

    speakable animal2 = Cat();
    std::cout << animal2.speak() << std::endl;  // Meow!

    return 0;
}
```

---

## 运算符概念

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/operators.hpp>
#include <iostream>

namespace te = boost::type_erasure;

int main() {
    // 定义支持算术运算的类型
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            te::addable<>,
            te::subtractable<>,
            te::multipliable<>,
            te::ostreamable<>
        >
    > arithmetic;

    arithmetic x = 10;
    arithmetic y = 20;

    arithmetic sum = x + y;
    arithmetic diff = y - x;
    arithmetic prod = x * y;

    std::cout << "Sum: " << sum << std::endl;       // 30
    std::cout << "Diff: " << diff << std::endl;     // 10
    std::cout << "Product: " << prod << std::endl;  // 200

    // 可以改变存储的类型
    x = 5.5;
    y = 2.5;
    std::cout << "Double sum: " << (x + y) << std::endl;  // 8.0

    return 0;
}
```

---

## 可调用对象

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/callable.hpp>
#include <iostream>
#include <functional>

namespace te = boost::type_erasure;

int main() {
    // 定义可调用类型（类似 std::function）
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            te::callable<int(int, int)>
        >
    > function_type;

    // Lambda
    function_type f1 = [](int a, int b) { return a + b; };
    std::cout << "Lambda: " << f1(3, 4) << std::endl;  // 7

    // 函数指针
    function_type f2 = [](int a, int b) { return a * b; };
    std::cout << "Multiply: " << f2(3, 4) << std::endl;  // 12

    // std::function
    function_type f3 = std::function<int(int, int)>([](int a, int b) { return a - b; });
    std::cout << "Subtract: " << f3(10, 3) << std::endl;  // 7

    return 0;
}
```

---

## 迭代器概念

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/operators.hpp>
#include <boost/type_erasure/iterator.hpp>
#include <iostream>
#include <vector>
#include <list>

namespace te = boost::type_erasure;

int main() {
    // 定义前向迭代器
    typedef te::any<
        boost::mpl::vector<
            te::forward_iterator<>,
            te::same_type<te::forward_iterator<>::value_type, int>
        >,
        te::_iter
    > forward_iterator_any;

    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::list<int> lst = {10, 20, 30};

    // 使用 vector 迭代器
    forward_iterator_any it1 = vec.begin();
    forward_iterator_any end1 = vec.end();

    std::cout << "Vector: ";
    while (it1 != end1) {
        std::cout << *it1 << " ";
        ++it1;
    }
    std::cout << std::endl;

    // 使用 list 迭代器
    forward_iterator_any it2 = lst.begin();
    forward_iterator_any end2 = lst.end();

    std::cout << "List: ";
    while (it2 != end2) {
        std::cout << *it2 << " ";
        ++it2;
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 组合多个概念

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/operators.hpp>
#include <boost/type_erasure/member.hpp>
#include <iostream>
#include <string>

namespace te = boost::type_erasure;

// 定义自定义成员函数
BOOST_TYPE_ERASURE_MEMBER((has_to_string), to_string, 0)
BOOST_TYPE_ERASURE_MEMBER((has_size), size, 0)

class MyClass {
public:
    MyClass(int val) : value_(val) {}

    std::string to_string() const {
        return "Value: " + std::to_string(value_);
    }

    size_t size() const {
        return sizeof(value_);
    }

    MyClass operator+(const MyClass& other) const {
        return MyClass(value_ + other.value_);
    }

private:
    int value_;
};

int main() {
    // 组合多个概念
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            te::addable<>,
            has_to_string<std::string(), const te::_self>,
            has_size<size_t(), const te::_self>
        >
    > complex_type;

    complex_type obj1 = MyClass(10);
    complex_type obj2 = MyClass(20);

    std::cout << obj1.to_string() << std::endl;  // Value: 10
    std::cout << "Size: " << obj1.size() << std::endl;

    complex_type sum = obj1 + obj2;
    std::cout << sum.to_string() << std::endl;   // Value: 30

    return 0;
}
```

---

## 与多态的比较

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/member.hpp>
#include <iostream>
#include <memory>
#include <vector>

namespace te = boost::type_erasure;

BOOST_TYPE_ERASURE_MEMBER((has_draw), draw, 0)

// 传统多态方式
class ShapeBase {
public:
    virtual ~ShapeBase() = default;
    virtual void draw() const = 0;
};

class Circle : public ShapeBase {
public:
    void draw() const override {
        std::cout << "Drawing circle (inheritance)" << std::endl;
    }
};

class Rectangle : public ShapeBase {
public:
    void draw() const override {
        std::cout << "Drawing rectangle (inheritance)" << std::endl;
    }
};

// 类型擦除方式（无需继承）
class CircleTE {
public:
    void draw() const {
        std::cout << "Drawing circle (type erasure)" << std::endl;
    }
};

class RectangleTE {
public:
    void draw() const {
        std::cout << "Drawing rectangle (type erasure)" << std::endl;
    }
};

int main() {
    // 传统多态
    std::cout << "Traditional polymorphism:\n";
    std::vector<std::unique_ptr<ShapeBase>> shapes1;
    shapes1.push_back(std::make_unique<Circle>());
    shapes1.push_back(std::make_unique<Rectangle>());

    for (const auto& shape : shapes1) {
        shape->draw();
    }

    // 类型擦除
    std::cout << "\nType erasure:\n";

    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            has_draw<void(), const te::_self>
        >
    > drawable;

    std::vector<drawable> shapes2;
    shapes2.push_back(CircleTE());
    shapes2.push_back(RectangleTE());

    for (const auto& shape : shapes2) {
        shape.draw();
    }

    return 0;
}
```

**优势**:
- 无需修改现有类（非侵入式）
- 更灵活的组合概念
- 可以在运行时改变类型

---

## 类型查询和转换

```cpp
#include <boost/type_erasure/any.hpp>
#include <boost/type_erasure/builtin.hpp>
#include <boost/type_erasure/any_cast.hpp>
#include <iostream>
#include <string>

namespace te = boost::type_erasure;

int main() {
    typedef te::any<
        boost::mpl::vector<
            te::copy_constructible<>,
            te::typeid_<>
        >
    > any_type;

    any_type x = 42;

    // 类型查询
    std::cout << "Type: " << te::typeid_of(x).name() << std::endl;

    // 类型转换
    try {
        int value = te::any_cast<int>(x);
        std::cout << "Value as int: " << value << std::endl;
    } catch (const te::bad_any_cast& e) {
        std::cout << "Cast failed: " << e.what() << std::endl;
    }

    // 改变类型
    x = std::string("Hello");
    std::cout << "Type: " << te::typeid_of(x).name() << std::endl;

    std::string str = te::any_cast<std::string>(x);
    std::cout << "Value as string: " << str << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.TypeErasure 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_typeerasure.html)
- [Boost.TypeErasure 教程](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_typeerasure/tutorial.html)

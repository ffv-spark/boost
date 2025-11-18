# Boost.Python - Python绑定库

## 概述

Boost.Python 提供在 C++ 和 Python 之间无缝互操作的功能，可以轻松地将 C++ 类和函数暴露给 Python。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/python.hpp>

// C++ 函数
const char* greet() {
    return "Hello from C++!";
}

BOOST_PYTHON_MODULE(hello) {
    using namespace boost::python;
    def("greet", greet);
}
```

**编译**: `g++ -std=c++14 -shared -fPIC hello.cpp -o hello.so -lboost_python39 -lpython3.9`

**Python 使用**:
```python
import hello
print(hello.greet())  # 输出: Hello from C++!
```

---

## 导出函数

```cpp
#include <boost/python.hpp>

int add(int a, int b) {
    return a + b;
}

double multiply(double a, double b) {
    return a * b;
}

std::string concatenate(const std::string& a, const std::string& b) {
    return a + b;
}

BOOST_PYTHON_MODULE(math_ops) {
    using namespace boost::python;
    
    def("add", add);
    def("multiply", multiply);
    def("concatenate", concatenate);
}
```

**Python 使用**:
```python
import math_ops
print(math_ops.add(2, 3))              # 5
print(math_ops.multiply(2.5, 3.0))     # 7.5
print(math_ops.concatenate("Hello", " World"))  # Hello World
```

---

## 导出类

```cpp
#include <boost/python.hpp>
#include <string>

class Person {
public:
    Person(const std::string& name, int age) 
        : name_(name), age_(age) {}
    
    std::string get_name() const { return name_; }
    int get_age() const { return age_; }
    
    void set_age(int age) { age_ = age; }
    
    std::string introduce() const {
        return "My name is " + name_ + ", I'm " + std::to_string(age_);
    }

private:
    std::string name_;
    int age_;
};

BOOST_PYTHON_MODULE(person) {
    using namespace boost::python;
    
    class_<Person>("Person", init<std::string, int>())
        .def("get_name", &Person::get_name)
        .def("get_age", &Person::get_age)
        .def("set_age", &Person::set_age)
        .def("introduce", &Person::introduce);
}
```

**Python 使用**:
```python
import person
p = person.Person("Alice", 30)
print(p.get_name())    # Alice
print(p.introduce())   # My name is Alice, I'm 30
p.set_age(31)
print(p.get_age())     # 31
```

---

## 属性导出

```cpp
#include <boost/python.hpp>

class Rectangle {
public:
    Rectangle(double w, double h) : width_(w), height_(h) {}
    
    double get_width() const { return width_; }
    void set_width(double w) { width_ = w; }
    
    double get_height() const { return height_; }
    void set_height(double h) { height_ = h; }
    
    double area() const { return width_ * height_; }

private:
    double width_, height_;
};

BOOST_PYTHON_MODULE(shapes) {
    using namespace boost::python;
    
    class_<Rectangle>("Rectangle", init<double, double>())
        .add_property("width", &Rectangle::get_width, &Rectangle::set_width)
        .add_property("height", &Rectangle::get_height, &Rectangle::set_height)
        .add_property("area", &Rectangle::area)  // 只读属性
        .def("area", &Rectangle::area);
}
```

**Python 使用**:
```python
import shapes
r = shapes.Rectangle(5.0, 3.0)
print(r.width)   # 5.0
print(r.area)    # 15.0
r.width = 10.0
print(r.area)    # 30.0
```

---

## 运算符重载

```cpp
#include <boost/python.hpp>

class Vector2D {
public:
    Vector2D(double x, double y) : x_(x), y_(y) {}
    
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x_ + other.x_, y_ + other.y_);
    }
    
    Vector2D operator*(double scalar) const {
        return Vector2D(x_ * scalar, y_ * scalar);
    }
    
    std::string to_string() const {
        return "(" + std::to_string(x_) + ", " + std::to_string(y_) + ")";
    }
    
    double x_, y_;
};

BOOST_PYTHON_MODULE(vector) {
    using namespace boost::python;
    
    class_<Vector2D>("Vector2D", init<double, double>())
        .def_readwrite("x", &Vector2D::x_)
        .def_readwrite("y", &Vector2D::y_)
        .def(self + self)  // __add__
        .def(self * double())  // __mul__
        .def("__str__", &Vector2D::to_string);
}
```

**Python 使用**:
```python
import vector
v1 = vector.Vector2D(1.0, 2.0)
v2 = vector.Vector2D(3.0, 4.0)
v3 = v1 + v2
print(v3)  # (4.0, 6.0)
v4 = v1 * 2.0
print(v4)  # (2.0, 4.0)
```

---

## 继承

```cpp
#include <boost/python.hpp>
#include <string>

class Animal {
public:
    virtual ~Animal() {}
    virtual std::string speak() const = 0;
};

class Dog : public Animal {
public:
    std::string speak() const override {
        return "Woof!";
    }
};

class Cat : public Animal {
public:
    std::string speak() const override {
        return "Meow!";
    }
};

BOOST_PYTHON_MODULE(animals) {
    using namespace boost::python;
    
    class_<Animal, boost::noncopyable>("Animal", no_init)
        .def("speak", pure_virtual(&Animal::speak));
    
    class_<Dog, bases<Animal>>("Dog")
        .def("speak", &Dog::speak);
    
    class_<Cat, bases<Animal>>("Cat")
        .def("speak", &Cat::speak);
}
```

**Python 使用**:
```python
import animals
dog = animals.Dog()
cat = animals.Cat()
print(dog.speak())  # Woof!
print(cat.speak())  # Meow!
```

---

## 默认参数

```cpp
#include <boost/python.hpp>
#include <string>

std::string greet(const std::string& name = "World", int times = 1) {
    std::string result;
    for (int i = 0; i < times; ++i) {
        result += "Hello, " + name + "! ";
    }
    return result;
}

BOOST_PYTHON_MODULE(greetings) {
    using namespace boost::python;
    
    def("greet", greet,
        (arg("name") = "World", arg("times") = 1));
}
```

**Python 使用**:
```python
import greetings
print(greetings.greet())                    # Hello, World! 
print(greetings.greet("Alice"))             # Hello, Alice! 
print(greetings.greet("Bob", 2))            # Hello, Bob! Hello, Bob!
print(greetings.greet(times=3, name="Eve")) # Hello, Eve! Hello, Eve! Hello, Eve!
```

---

## 列表和字典

```cpp
#include <boost/python.hpp>
#include <vector>
#include <map>

std::vector<int> get_numbers() {
    return {1, 2, 3, 4, 5};
}

int sum_list(const boost::python::list& numbers) {
    int total = 0;
    for (int i = 0; i < len(numbers); ++i) {
        total += boost::python::extract<int>(numbers[i]);
    }
    return total;
}

boost::python::dict get_scores() {
    boost::python::dict d;
    d["Alice"] = 95;
    d["Bob"] = 87;
    d["Charlie"] = 92;
    return d;
}

BOOST_PYTHON_MODULE(collections) {
    using namespace boost::python;
    
    def("get_numbers", get_numbers);
    def("sum_list", sum_list);
    def("get_scores", get_scores);
}
```

**Python 使用**:
```python
import collections
print(collections.get_numbers())      # [1, 2, 3, 4, 5]
print(collections.sum_list([10, 20, 30]))  # 60
print(collections.get_scores())       # {'Alice': 95, 'Bob': 87, 'Charlie': 92}
```

---

## 异常处理

```cpp
#include <boost/python.hpp>
#include <stdexcept>

void risky_function(int value) {
    if (value < 0) {
        throw std::invalid_argument("Value must be non-negative");
    }
    if (value == 0) {
        throw std::runtime_error("Value cannot be zero");
    }
}

BOOST_PYTHON_MODULE(errors) {
    using namespace boost::python;
    
    def("risky_function", risky_function);
    
    // 注册异常转换
    register_exception_translator<std::invalid_argument>(
        [](const std::invalid_argument& e) {
            PyErr_SetString(PyExc_ValueError, e.what());
        });
}
```

**Python 使用**:
```python
import errors
try:
    errors.risky_function(-1)
except ValueError as e:
    print(f"Caught: {e}")  # Caught: Value must be non-negative
```

---

## 回调函数

```cpp
#include <boost/python.hpp>

void call_python_function(boost::python::object func, int value) {
    func(value);
}

int apply_operation(int a, int b, boost::python::object op) {
    return boost::python::extract<int>(op(a, b));
}

BOOST_PYTHON_MODULE(callbacks) {
    using namespace boost::python;
    
    def("call_python_function", call_python_function);
    def("apply_operation", apply_operation);
}
```

**Python 使用**:
```python
import callbacks

def my_print(x):
    print(f"Value is: {x}")

callbacks.call_python_function(my_print, 42)  # Value is: 42

result = callbacks.apply_operation(5, 3, lambda x, y: x + y)
print(result)  # 8
```

---

## 参考资源

- [Boost.Python 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/python/doc/html/index.html)
- [Python/C++ 绑定教程](https://www.boost.org/doc/libs/1_90_0/libs/python/doc/html/tutorial/index.html)

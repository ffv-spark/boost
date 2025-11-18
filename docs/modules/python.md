# Boost.Python - Python 绑定库

## 概述

Boost.Python 允许在 C++ 和 Python 之间无缝集成，可以从 Python 调用 C++ 代码，也可以从 C++ 调用 Python。

**类型**: 需要编译链接的库

**链接库**: `-lboost_python3x` (x 为 Python 版本号，如 `-lboost_python39`)

**依赖**: Python 开发库

---

## 快速开始 - 暴露 C++ 函数

```cpp
#include <boost/python.hpp>
#include <string>

// 要暴露的 C++ 函数
std::string greet(const std::string& name) {
    return "Hello, " + name + "!";
}

int add(int a, int b) {
    return a + b;
}

// 定义 Python 模块
BOOST_PYTHON_MODULE(example) {
    using namespace boost::python;

    // 暴露函数
    def("greet", greet);
    def("add", add);
}
```

**编译**:
```bash
g++ -std=c++11 -shared -fPIC \
    example.cpp \
    -o example.so \
    -lboost_python39 \
    $(python3-config --includes --ldflags)
```

**Python 使用**:
```python
import example

print(example.greet("World"))  # Hello, World!
print(example.add(3, 4))       # 7
```

---

## 暴露 C++ 类

```cpp
#include <boost/python.hpp>
#include <string>

class Person {
public:
    Person(const std::string& name, int age)
        : name_(name), age_(age) {}

    std::string get_name() const { return name_; }
    void set_name(const std::string& name) { name_ = name; }

    int get_age() const { return age_; }
    void set_age(int age) { age_ = age; }

    std::string introduce() const {
        return "I'm " + name_ + ", " + std::to_string(age_) + " years old.";
    }

private:
    std::string name_;
    int age_;
};

BOOST_PYTHON_MODULE(person_module) {
    using namespace boost::python;

    class_<Person>("Person", init<std::string, int>())
        .def("get_name", &Person::get_name)
        .def("set_name", &Person::set_name)
        .def("get_age", &Person::get_age)
        .def("set_age", &Person::set_age)
        .def("introduce", &Person::introduce)
        // 添加属性
        .add_property("name", &Person::get_name, &Person::set_name)
        .add_property("age", &Person::get_age, &Person::set_age);
}
```

**Python 使用**:
```python
import person_module

p = person_module.Person("Alice", 30)
print(p.introduce())  # I'm Alice, 30 years old.

p.name = "Bob"
p.age = 25
print(p.introduce())  # I'm Bob, 25 years old.
```

---

## 类继承和虚函数

```cpp
#include <boost/python.hpp>
#include <string>

// C++ 基类
class Animal {
public:
    virtual ~Animal() {}
    virtual std::string speak() const = 0;
    std::string get_type() const { return "Animal"; }
};

// Python 包装类
class AnimalWrap : public Animal, public boost::python::wrapper<Animal> {
public:
    std::string speak() const override {
        return this->get_override("speak")();
    }
};

// 派生类
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

// 使用多态的函数
std::string make_speak(const Animal& animal) {
    return animal.speak();
}

BOOST_PYTHON_MODULE(animals) {
    using namespace boost::python;

    class_<AnimalWrap, boost::noncopyable>("Animal", no_init)
        .def("speak", pure_virtual(&Animal::speak))
        .def("get_type", &Animal::get_type);

    class_<Dog, bases<Animal>>("Dog")
        .def("speak", &Dog::speak);

    class_<Cat, bases<Animal>>("Cat")
        .def("speak", &Cat::speak);

    def("make_speak", make_speak);
}
```

**Python 使用**:
```python
import animals

# 使用 C++ 类
dog = animals.Dog()
cat = animals.Cat()

print(dog.speak())  # Woof!
print(cat.speak())  # Meow!

# 多态
print(animals.make_speak(dog))  # Woof!
print(animals.make_speak(cat))  # Meow!

# 从 Python 继承
class Bird(animals.Animal):
    def speak(self):
        return "Tweet!"

bird = Bird()
print(animals.make_speak(bird))  # Tweet!
```

---

## 处理容器（vector, list, map）

```cpp
#include <boost/python.hpp>
#include <boost/python/suite/indexing/vector_indexing_suite.hpp>
#include <boost/python/suite/indexing/map_indexing_suite.hpp>
#include <vector>
#include <map>
#include <string>

typedef std::vector<int> IntVector;
typedef std::map<std::string, int> StringIntMap;

IntVector create_vector() {
    return IntVector{1, 2, 3, 4, 5};
}

StringIntMap create_map() {
    return StringIntMap{{"one", 1}, {"two", 2}, {"three", 3}};
}

BOOST_PYTHON_MODULE(containers) {
    using namespace boost::python;

    // 暴露 vector
    class_<IntVector>("IntVector")
        .def(vector_indexing_suite<IntVector>());

    // 暴露 map
    class_<StringIntMap>("StringIntMap")
        .def(map_indexing_suite<StringIntMap>());

    def("create_vector", create_vector);
    def("create_map", create_map);
}
```

**Python 使用**:
```python
import containers

# Vector 操作
vec = containers.create_vector()
print(len(vec))      # 5
print(vec[0])        # 1
vec.append(6)
print(list(vec))     # [1, 2, 3, 4, 5, 6]

# Map 操作
m = containers.create_map()
print(m["one"])      # 1
m["four"] = 4
print(dict(m))       # {'one': 1, 'two': 2, 'three': 3, 'four': 4}
```

---

## 异常处理

```cpp
#include <boost/python.hpp>
#include <stdexcept>
#include <string>

class MyException : public std::runtime_error {
public:
    MyException(const std::string& msg) : std::runtime_error(msg) {}
};

void throw_exception() {
    throw MyException("Something went wrong!");
}

int safe_divide(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("Division by zero!");
    }
    return a / b;
}

// 将 C++ 异常转换为 Python 异常
void translate_exception(const MyException& e) {
    PyErr_SetString(PyExc_RuntimeError, e.what());
}

BOOST_PYTHON_MODULE(exceptions) {
    using namespace boost::python;

    // 注册异常转换器
    register_exception_translator<MyException>(translate_exception);

    def("throw_exception", throw_exception);
    def("safe_divide", safe_divide);
}
```

**Python 使用**:
```python
import exceptions

try:
    exceptions.throw_exception()
except RuntimeError as e:
    print(f"Caught: {e}")

try:
    result = exceptions.safe_divide(10, 0)
except ValueError as e:
    print(f"Caught: {e}")
```

---

## 默认参数和重载

```cpp
#include <boost/python.hpp>
#include <string>

std::string greet(const std::string& name, const std::string& greeting = "Hello") {
    return greeting + ", " + name + "!";
}

int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }

// 函数重载需要指定类型
int (*add_int)(int, int) = &add;
double (*add_double)(double, double) = &add;

BOOST_PYTHON_MODULE(defaults) {
    using namespace boost::python;

    // 默认参数
    def("greet", greet, (arg("name"), arg("greeting") = "Hello"));

    // 函数重载
    def("add_int", add_int);
    def("add_double", add_double);
}
```

**Python 使用**:
```python
import defaults

print(defaults.greet("World"))                  # Hello, World!
print(defaults.greet("World", "Hi"))            # Hi, World!

print(defaults.add_int(3, 4))                   # 7
print(defaults.add_double(3.5, 4.2))            # 7.7
```

---

## 从 C++ 调用 Python

```cpp
#include <boost/python.hpp>
#include <iostream>

void call_python_function() {
    try {
        // 获取主模块
        namespace py = boost::python;
        py::object main = py::import("__main__");
        py::object global = main.attr("__dict__");

        // 执行 Python 代码
        py::exec("def hello(name):\n"
                "    return f'Hello, {name}!'",
                global, global);

        // 调用 Python 函数
        py::object hello_func = global["hello"];
        py::object result = hello_func("C++");

        // 转换结果
        std::string msg = py::extract<std::string>(result);
        std::cout << msg << std::endl;

    } catch (const boost::python::error_already_set&) {
        PyErr_Print();
    }
}

BOOST_PYTHON_MODULE(call_python) {
    using namespace boost::python;
    def("call_python_function", call_python_function);
}
```

---

## NumPy 数组支持

```cpp
#include <boost/python.hpp>
#include <boost/python/numpy.hpp>
#include <iostream>

namespace py = boost::python;
namespace np = boost::python::numpy;

// 处理 NumPy 数组
np::ndarray square_array(const np::ndarray& arr) {
    // 获取数组信息
    int ndim = arr.get_nd();
    const Py_intptr_t* shape = arr.get_shape();

    std::cout << "Array shape: ";
    for (int i = 0; i < ndim; ++i) {
        std::cout << shape[i] << " ";
    }
    std::cout << std::endl;

    // 创建结果数组
    np::ndarray result = np::zeros(ndim, shape, np::dtype::get_builtin<double>());

    // 处理数据（简化示例）
    const double* input_data = reinterpret_cast<const double*>(arr.get_data());
    double* output_data = reinterpret_cast<double*>(result.get_data());

    int size = 1;
    for (int i = 0; i < ndim; ++i) {
        size *= shape[i];
    }

    for (int i = 0; i < size; ++i) {
        output_data[i] = input_data[i] * input_data[i];
    }

    return result;
}

BOOST_PYTHON_MODULE(numpy_example) {
    // 初始化 NumPy
    np::initialize();

    py::def("square_array", square_array);
}
```

**Python 使用**:
```python
import numpy as np
import numpy_example

arr = np.array([1.0, 2.0, 3.0, 4.0])
result = numpy_example.square_array(arr)
print(result)  # [1. 4. 9. 16.]
```

---

## 参考资源

- [Boost.Python 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/python/doc/html/index.html)
- [Boost.Python 教程](https://www.boost.org/doc/libs/1_90_0/libs/python/doc/html/tutorial/index.html)

# Boost.TypeTraits - 类型特性库

## 概述

Boost.TypeTraits 提供编译期类型查询和转换功能，是模板元编程的基础工具。

**类型**: 仅头文件库

**注意**: C++11 引入了 `<type_traits>`，建议优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    // 检查是否为整数类型
    std::cout << "int 是整数: "
              << boost::is_integral<int>::value << std::endl;

    std::cout << "double 是整数: "
              << boost::is_integral<double>::value << std::endl;

    // 检查是否为指针
    std::cout << "int* 是指针: "
              << boost::is_pointer<int*>::value << std::endl;

    std::cout << "int 是指针: "
              << boost::is_pointer<int>::value << std::endl;

    return 0;
}
```

---

## 基本类型检测

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

template<typename T>
void check_type() {
    std::cout << "\n类型检查:\n";

    std::cout << "是否为整数: " << boost::is_integral<T>::value << std::endl;
    std::cout << "是否为浮点数: " << boost::is_floating_point<T>::value << std::endl;
    std::cout << "是否为算术类型: " << boost::is_arithmetic<T>::value << std::endl;
    std::cout << "是否为指针: " << boost::is_pointer<T>::value << std::endl;
    std::cout << "是否为引用: " << boost::is_reference<T>::value << std::endl;
    std::cout << "是否为数组: " << boost::is_array<T>::value << std::endl;
    std::cout << "是否为类: " << boost::is_class<T>::value << std::endl;
    std::cout << "是否为函数: " << boost::is_function<T>::value << std::endl;
}

class MyClass {};

int main() {
    check_type<int>();
    check_type<double>();
    check_type<int*>();
    check_type<int&>();
    check_type<int[]>();
    check_type<MyClass>();

    return 0;
}
```

---

## 类型修饰符检测

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

template<typename T>
void check_modifiers() {
    std::cout << "\n修饰符检查:\n";

    std::cout << "是否为 const: " << boost::is_const<T>::value << std::endl;
    std::cout << "是否为 volatile: " << boost::is_volatile<T>::value << std::endl;
    std::cout << "是否为 signed: " << boost::is_signed<T>::value << std::endl;
    std::cout << "是否为 unsigned: " << boost::is_unsigned<T>::value << std::endl;
}

int main() {
    check_modifiers<int>();
    check_modifiers<const int>();
    check_modifiers<volatile int>();
    check_modifiers<unsigned int>();

    return 0;
}
```

---

## 类型关系检测

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

class Base {};
class Derived : public Base {};
class Unrelated {};

int main() {
    // 相同类型检测
    std::cout << "int 和 int 相同: "
              << boost::is_same<int, int>::value << std::endl;

    std::cout << "int 和 long 相同: "
              << boost::is_same<int, long>::value << std::endl;

    // 基类派生类检测
    std::cout << "Derived 继承自 Base: "
              << boost::is_base_of<Base, Derived>::value << std::endl;

    std::cout << "Unrelated 继承自 Base: "
              << boost::is_base_of<Base, Unrelated>::value << std::endl;

    // 类型转换检测
    std::cout << "int 可转换为 double: "
              << boost::is_convertible<int, double>::value << std::endl;

    std::cout << "int* 可转换为 void*: "
              << boost::is_convertible<int*, void*>::value << std::endl;

    return 0;
}
```

---

## 类型转换

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

template<typename T>
void print_transformed_types() {
    typedef typename boost::remove_const<T>::type non_const;
    typedef typename boost::remove_pointer<T>::type non_pointer;
    typedef typename boost::remove_reference<T>::type non_reference;
    typedef typename boost::add_pointer<T>::type with_pointer;
    typedef typename boost::add_const<T>::type with_const;

    std::cout << "\n类型转换示例:\n";
    std::cout << "原类型是 const: " << boost::is_const<T>::value << std::endl;
    std::cout << "移除 const 后是 const: " << boost::is_const<non_const>::value << std::endl;

    std::cout << "原类型是指针: " << boost::is_pointer<T>::value << std::endl;
    std::cout << "移除指针后是指针: " << boost::is_pointer<non_pointer>::value << std::endl;

    std::cout << "原类型是引用: " << boost::is_reference<T>::value << std::endl;
    std::cout << "移除引用后是引用: " << boost::is_reference<non_reference>::value << std::endl;
}

int main() {
    print_transformed_types<const int*>();
    print_transformed_types<int&>();

    return 0;
}
```

---

## 条件类型选择

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

// 根据条件选择类型
template<bool Condition, typename T, typename U>
struct conditional {
    typedef typename boost::conditional<Condition, T, U>::type type;
};

int main() {
    // 如果条件为真，选择 int，否则选择 double
    typedef conditional<true, int, double>::type Type1;
    typedef conditional<false, int, double>::type Type2;

    std::cout << "Type1 是 int: " << boost::is_same<Type1, int>::value << std::endl;
    std::cout << "Type2 是 double: " << boost::is_same<Type2, double>::value << std::endl;

    // 实用例子：选择较大的类型
    typedef typename boost::conditional<
        (sizeof(int) > sizeof(long)),
        int,
        long
    >::type LargerType;

    std::cout << "较大类型的大小: " << sizeof(LargerType) << std::endl;

    return 0;
}
```

---

## 函数特性检测

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int add(int a, int b) { return a + b; }

class Calculator {
public:
    int multiply(int a, int b) { return a * b; }
    static int divide(int a, int b) { return a / b; }
};

int main() {
    // 函数指针
    std::cout << "add 是函数: "
              << boost::is_function<decltype(add)>::value << std::endl;

    std::cout << "add 指针是函数指针: "
              << boost::is_function<int(*)(int, int)>::value << std::endl;

    // 成员函数指针
    typedef int (Calculator::*MemberFunc)(int, int);
    std::cout << "成员函数指针是成员函数指针: "
              << boost::is_member_function_pointer<MemberFunc>::value << std::endl;

    return 0;
}
```

---

## SFINAE 技术应用

```cpp
#include <boost/type_traits.hpp>
#include <boost/utility/enable_if.hpp>
#include <iostream>
#include <vector>

// 只为整数类型启用
template<typename T>
typename boost::enable_if<boost::is_integral<T>, T>::type
square(T value) {
    return value * value;
}

// 只为浮点类型启用
template<typename T>
typename boost::enable_if<boost::is_floating_point<T>, T>::type
square(T value) {
    std::cout << "(浮点版本) ";
    return value * value;
}

// 只为容器类型启用（有 value_type 成员）
template<typename T>
typename boost::enable_if<boost::has_value_type<T>, size_t>::type
get_size(const T& container) {
    return container.size();
}

int main() {
    std::cout << "整数平方: " << square(5) << std::endl;
    std::cout << "浮点数平方: " << square(3.14) << std::endl;

    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::cout << "容器大小: " << get_size(vec) << std::endl;

    return 0;
}
```

---

## 类属性检测

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

class Abstract {
public:
    virtual void pure_virtual() = 0;
};

class Concrete {
public:
    Concrete() {}
    Concrete(const Concrete&) {}
    virtual ~Concrete() {}
};

class Empty {};

int main() {
    std::cout << "Abstract 是抽象类: "
              << boost::is_abstract<Abstract>::value << std::endl;

    std::cout << "Concrete 是抽象类: "
              << boost::is_abstract<Concrete>::value << std::endl;

    std::cout << "Empty 是空类: "
              << boost::is_empty<Empty>::value << std::endl;

    std::cout << "Concrete 有虚析构: "
              << boost::has_virtual_destructor<Concrete>::value << std::endl;

    std::cout << "Concrete 是 POD: "
              << boost::is_pod<Concrete>::value << std::endl;

    return 0;
}
```

---

## 对齐相关

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << "char 对齐: "
              << boost::alignment_of<char>::value << std::endl;

    std::cout << "int 对齐: "
              << boost::alignment_of<int>::value << std::endl;

    std::cout << "double 对齐: "
              << boost::alignment_of<double>::value << std::endl;

    struct S {
        char c;
        int i;
        double d;
    };

    std::cout << "S 对齐: "
              << boost::alignment_of<S>::value << std::endl;

    std::cout << "S 大小: " << sizeof(S) << std::endl;

    return 0;
}
```

---

## 类型萃取实用工具

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

template<typename T>
struct TypeInfo {
    static void print() {
        std::cout << "\n=== 类型信息 ===\n";
        std::cout << "大小: " << sizeof(T) << " 字节\n";
        std::cout << "对齐: " << boost::alignment_of<T>::value << " 字节\n";
        std::cout << "是POD: " << boost::is_pod<T>::value << "\n";
        std::cout << "是空类: " << boost::is_empty<T>::value << "\n";
        std::cout << "是多态: " << boost::is_polymorphic<T>::value << "\n";
        std::cout << "有默认构造: " << boost::has_trivial_constructor<T>::value << "\n";
        std::cout << "有默认析构: " << boost::has_trivial_destructor<T>::value << "\n";
    }
};

class MyClass {
public:
    virtual ~MyClass() {}
    int data;
};

int main() {
    TypeInfo<int>::print();
    TypeInfo<MyClass>::print();

    return 0;
}
```

---

## 参考资源

- [Boost.TypeTraits 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/type_traits/doc/html/index.html)
- [C++11 type_traits](https://en.cppreference.com/w/cpp/header/type_traits)

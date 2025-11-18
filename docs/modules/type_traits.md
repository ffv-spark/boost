# Boost.TypeTraits - 类型特性库

## 概述

Boost.TypeTraits 提供编译期类型信息查询，是 C++11 `<type_traits>` 的前身。

**类型**: 仅头文件库

**注意**: C++11 引入了 `<type_traits>`，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    std::cout << "int 是整数: " << boost::is_integral<int>::value << std::endl;
    std::cout << "double 是整数: " << boost::is_integral<double>::value << std::endl;
    std::cout << "int* 是指针: " << boost::is_pointer<int*>::value << std::endl;
    
    return 0;
}
```

---

## 基本类型检查

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    
    // 整数类型
    std::cout << "int 是整数: " << boost::is_integral<int>::value << std::endl;
    std::cout << "char 是整数: " << boost::is_integral<char>::value << std::endl;
    
    // 浮点类型
    std::cout << "float 是浮点: " << boost::is_floating_point<float>::value << std::endl;
    std::cout << "double 是浮点: " << boost::is_floating_point<double>::value << std::endl;
    
    // 算术类型
    std::cout << "int 是算术: " << boost::is_arithmetic<int>::value << std::endl;
    std::cout << "double 是算术: " << boost::is_arithmetic<double>::value << std::endl;
    
    return 0;
}
```

---

## 复合类型检查

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    
    // 指针
    std::cout << "int* 是指针: " << boost::is_pointer<int*>::value << std::endl;
    std::cout << "int 是指针: " << boost::is_pointer<int>::value << std::endl;
    
    // 引用
    std::cout << "int& 是引用: " << boost::is_reference<int&>::value << std::endl;
    std::cout << "int 是引用: " << boost::is_reference<int>::value << std::endl;
    
    // 数组
    std::cout << "int[10] 是数组: " << boost::is_array<int[10]>::value << std::endl;
    std::cout << "int* 是数组: " << boost::is_array<int*>::value << std::endl;
    
    return 0;
}
```

---

## const/volatile 检查

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    
    // const
    std::cout << "const int 是const: " 
              << boost::is_const<const int>::value << std::endl;
    std::cout << "int 是const: " 
              << boost::is_const<int>::value << std::endl;
    
    // volatile
    std::cout << "volatile int 是volatile: " 
              << boost::is_volatile<volatile int>::value << std::endl;
    std::cout << "int 是volatile: " 
              << boost::is_volatile<int>::value << std::endl;
    
    return 0;
}
```

---

## 类类型检查

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

struct MyStruct {};
class MyClass {};
union MyUnion { int x; double y; };
enum MyEnum { A, B, C };

int main() {
    std::cout << std::boolalpha;
    
    // 类
    std::cout << "MyClass 是类: " << boost::is_class<MyClass>::value << std::endl;
    std::cout << "MyStruct 是类: " << boost::is_class<MyStruct>::value << std::endl;
    std::cout << "int 是类: " << boost::is_class<int>::value << std::endl;
    
    // 联合
    std::cout << "MyUnion 是联合: " << boost::is_union<MyUnion>::value << std::endl;
    
    // 枚举
    std::cout << "MyEnum 是枚举: " << boost::is_enum<MyEnum>::value << std::endl;
    
    return 0;
}
```

---

## 函数类型检查

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

void func() {}
int func2(int x) { return x; }

int main() {
    std::cout << std::boolalpha;
    
    // 函数类型
    std::cout << "void() 是函数: " 
              << boost::is_function<decltype(func)>::value << std::endl;
    std::cout << "int(int) 是函数: " 
              << boost::is_function<decltype(func2)>::value << std::endl;
    
    // 函数指针
    typedef void (*func_ptr)();
    std::cout << "func_ptr 是指针: " 
              << boost::is_pointer<func_ptr>::value << std::endl;
    
    return 0;
}
```

---

## 类型转换

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    // 移除const
    typedef boost::remove_const<const int>::type int_type;
    std::cout << "移除const后是const: " 
              << boost::is_const<int_type>::value << std::endl;
    
    // 移除指针
    typedef boost::remove_pointer<int*>::type value_type;
    std::cout << "移除指针后是指针: " 
              << boost::is_pointer<value_type>::value << std::endl;
    
    // 移除引用
    typedef boost::remove_reference<int&>::type ref_type;
    std::cout << "移除引用后是引用: " 
              << boost::is_reference<ref_type>::value << std::endl;
    
    // 添加const
    typedef boost::add_const<int>::type const_type;
    std::cout << "添加const后是const: " 
              << boost::is_const<const_type>::value << std::endl;
    
    return 0;
}
```

---

## 对齐信息

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

struct alignas(16) Aligned16 {
    int data;
};

int main() {
    std::cout << "int 对齐: " 
              << boost::alignment_of<int>::value << std::endl;
    std::cout << "double 对齐: " 
              << boost::alignment_of<double>::value << std::endl;
    std::cout << "Aligned16 对齐: " 
              << boost::alignment_of<Aligned16>::value << std::endl;
    
    return 0;
}
```

---

## 条件类型

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

template <typename T>
void process() {
    // 根据条件选择类型
    typedef typename boost::conditional<
        boost::is_integral<T>::value,
        int,
        double
    >::type result_type;
    
    std::cout << "T = " << typeid(T).name() << ", "
              << "result_type = " << typeid(result_type).name() << std::endl;
}

int main() {
    process<int>();      // result_type = int
    process<double>();   // result_type = double
    process<char>();     // result_type = int
    
    return 0;
}
```

---

## 类型判断

```cpp
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    
    // 相同类型
    std::cout << "int 和 int 相同: " 
              << boost::is_same<int, int>::value << std::endl;
    std::cout << "int 和 long 相同: " 
              << boost::is_same<int, long>::value << std::endl;
    
    // 可转换
    std::cout << "int 可转换为 double: " 
              << boost::is_convertible<int, double>::value << std::endl;
    std::cout << "double 可转换为 int: " 
              << boost::is_convertible<double, int>::value << std::endl;
    
    return 0;
}
```

---

## SFINAE 应用

```cpp
#include <boost/type_traits.hpp>
#include <boost/utility/enable_if.hpp>
#include <iostream>

// 只对整数启用
template <typename T>
typename boost::enable_if<boost::is_integral<T>, void>::type
process(T value) {
    std::cout << "处理整数: " << value << std::endl;
}

// 只对浮点数启用
template <typename T>
typename boost::enable_if<boost::is_floating_point<T>, void>::type
process(T value) {
    std::cout << "处理浮点数: " << value << std::endl;
}

int main() {
    process(42);        // 处理整数
    process(3.14);      // 处理浮点数
    
    return 0;
}
```

---

## 参考资源

- [Boost.TypeTraits 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/type_traits/doc/html/index.html)
- [C++11 type_traits](https://en.cppreference.com/w/cpp/header/type_traits)

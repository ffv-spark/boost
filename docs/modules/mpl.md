# Boost.MPL - 元编程库

## 概述

Boost.MPL (Meta-Programming Library) 提供编译期元编程支持，可以操作类型序列和进行编译期计算。

**类型**: 仅头文件库

**难度**: 高级

---

## 快速开始 - 类型序列

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/at.hpp>
#include <boost/mpl/size.hpp>
#include <iostream>

namespace mpl = boost::mpl;

int main() {
    // 定义类型序列
    typedef mpl::vector<int, double, char, float> types;

    // 获取序列大小
    std::cout << "序列大小: " << mpl::size<types>::value << std::endl;

    // 访问特定位置的类型
    typedef mpl::at_c<types, 0>::type first_type;   // int
    typedef mpl::at_c<types, 2>::type third_type;   // char

    std::cout << "第一个类型大小: " << sizeof(first_type) << std::endl;
    std::cout << "第三个类型大小: " << sizeof(third_type) << std::endl;

    return 0;
}
```

---

## 编译期整数运算

```cpp
#include <boost/mpl/int.hpp>
#include <boost/mpl/plus.hpp>
#include <boost/mpl/minus.hpp>
#include <boost/mpl/times.hpp>
#include <boost/mpl/divides.hpp>
#include <iostream>

namespace mpl = boost::mpl;

int main() {
    // 定义编译期整数
    typedef mpl::int_<10> ten;
    typedef mpl::int_<20> twenty;

    // 编译期运算
    typedef mpl::plus<ten, twenty>::type sum;           // 30
    typedef mpl::minus<twenty, ten>::type difference;   // 10
    typedef mpl::times<ten, twenty>::type product;      // 200
    typedef mpl::divides<twenty, ten>::type quotient;   // 2

    std::cout << "10 + 20 = " << sum::value << std::endl;
    std::cout << "20 - 10 = " << difference::value << std::endl;
    std::cout << "10 * 20 = " << product::value << std::endl;
    std::cout << "20 / 10 = " << quotient::value << std::endl;

    return 0;
}
```

---

## 类型序列操作

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/push_back.hpp>
#include <boost/mpl/push_front.hpp>
#include <boost/mpl/pop_front.hpp>
#include <boost/mpl/front.hpp>
#include <boost/mpl/back.hpp>
#include <boost/mpl/size.hpp>
#include <iostream>

namespace mpl = boost::mpl;

int main() {
    typedef mpl::vector<int, double> vec1;

    // 在后面添加类型
    typedef mpl::push_back<vec1, char>::type vec2;  // vector<int, double, char>

    // 在前面添加类型
    typedef mpl::push_front<vec2, bool>::type vec3; // vector<bool, int, double, char>

    // 移除第一个类型
    typedef mpl::pop_front<vec3>::type vec4;        // vector<int, double, char>

    std::cout << "vec3 大小: " << mpl::size<vec3>::value << std::endl;
    std::cout << "vec4 大小: " << mpl::size<vec4>::value << std::endl;

    // 获取首尾类型
    typedef mpl::front<vec3>::type first;  // bool
    typedef mpl::back<vec3>::type last;    // char

    std::cout << "vec3 第一个类型大小: " << sizeof(first) << std::endl;
    std::cout << "vec3 最后类型大小: " << sizeof(last) << std::endl;

    return 0;
}
```

---

## 编译期条件判断

```cpp
#include <boost/mpl/if.hpp>
#include <boost/mpl/bool.hpp>
#include <boost/mpl/int.hpp>
#include <iostream>

namespace mpl = boost::mpl;

// 根据条件选择类型
template<bool UseDouble>
struct NumberType {
    typedef typename mpl::if_c<UseDouble, double, int>::type type;
};

int main() {
    typedef NumberType<true>::type Type1;   // double
    typedef NumberType<false>::type Type2;  // int

    Type1 v1 = 3.14;
    Type2 v2 = 42;

    std::cout << "Type1 值: " << v1 << std::endl;
    std::cout << "Type2 值: " << v2 << std::endl;

    // 使用 bool_
    typedef mpl::if_<mpl::true_, int, float>::type IntType;
    typedef mpl::if_<mpl::false_, int, float>::type FloatType;

    std::cout << "IntType 大小: " << sizeof(IntType) << std::endl;
    std::cout << "FloatType 大小: " << sizeof(FloatType) << std::endl;

    return 0;
}
```

---

## 编译期循环 - for_each

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/for_each.hpp>
#include <iostream>
#include <typeinfo>

namespace mpl = boost::mpl;

// 打印类型信息的函数对象
struct print_type {
    template<typename T>
    void operator()(T) const {
        std::cout << "类型: " << typeid(T).name()
                  << ", 大小: " << sizeof(T) << " 字节" << std::endl;
    }
};

int main() {
    typedef mpl::vector<char, short, int, long, float, double> types;

    std::cout << "遍历类型序列:\n";
    mpl::for_each<types>(print_type());

    return 0;
}
```

---

## 类型查找

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/find.hpp>
#include <boost/mpl/contains.hpp>
#include <boost/mpl/distance.hpp>
#include <boost/mpl/begin.hpp>
#include <iostream>

namespace mpl = boost::mpl;

int main() {
    typedef mpl::vector<int, double, char, float> types;

    // 检查是否包含某类型
    std::cout << "包含 double: "
              << mpl::contains<types, double>::value << std::endl;

    std::cout << "包含 long: "
              << mpl::contains<types, long>::value << std::endl;

    // 查找类型位置
    typedef mpl::find<types, char>::type iter;
    typedef mpl::distance<mpl::begin<types>::type, iter>::type position;

    std::cout << "char 的位置: " << position::value << std::endl;

    return 0;
}
```

---

## 类型过滤

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/copy_if.hpp>
#include <boost/mpl/back_inserter.hpp>
#include <boost/mpl/sizeof.hpp>
#include <boost/mpl/greater.hpp>
#include <boost/mpl/placeholders.hpp>
#include <boost/mpl/size.hpp>
#include <iostream>

namespace mpl = boost::mpl;
using namespace mpl::placeholders;

int main() {
    typedef mpl::vector<char, short, int, long, double> types;

    // 过滤：只保留大小大于2字节的类型
    typedef mpl::copy_if<
        types,
        mpl::greater<mpl::sizeof_<_1>, mpl::int_<2>>,
        mpl::back_inserter<mpl::vector<>>
    >::type large_types;

    std::cout << "原始类型数量: " << mpl::size<types>::value << std::endl;
    std::cout << "大类型数量: " << mpl::size<large_types>::value << std::endl;

    return 0;
}
```

---

## 类型转换

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/transform.hpp>
#include <boost/mpl/placeholders.hpp>
#include <boost/mpl/at.hpp>
#include <iostream>

namespace mpl = boost::mpl;
using namespace mpl::placeholders;

// 添加指针的元函数
template<typename T>
struct add_pointer {
    typedef T* type;
};

int main() {
    typedef mpl::vector<int, double, char> types;

    // 将所有类型转换为指针类型
    typedef mpl::transform<types, add_pointer<_1>>::type pointer_types;

    typedef mpl::at_c<pointer_types, 0>::type IntPtr;     // int*
    typedef mpl::at_c<pointer_types, 1>::type DoublePtr;  // double*
    typedef mpl::at_c<pointer_types, 2>::type CharPtr;    // char*

    std::cout << "IntPtr 大小: " << sizeof(IntPtr) << std::endl;
    std::cout << "DoublePtr 大小: " << sizeof(DoublePtr) << std::endl;
    std::cout << "CharPtr 大小: " << sizeof(CharPtr) << std::endl;

    return 0;
}
```

---

## 编译期阶乘

```cpp
#include <boost/mpl/int.hpp>
#include <boost/mpl/if.hpp>
#include <boost/mpl/times.hpp>
#include <boost/mpl/minus.hpp>
#include <iostream>

namespace mpl = boost::mpl;

// 编译期阶乘
template<int N>
struct factorial {
    typedef typename mpl::times<
        mpl::int_<N>,
        typename factorial<N - 1>::type
    >::type type;

    static const int value = type::value;
};

template<>
struct factorial<0> {
    typedef mpl::int_<1> type;
    static const int value = 1;
};

int main() {
    std::cout << "0! = " << factorial<0>::value << std::endl;
    std::cout << "5! = " << factorial<5>::value << std::endl;
    std::cout << "10! = " << factorial<10>::value << std::endl;

    return 0;
}
```

---

## 编译期列表处理

```cpp
#include <boost/mpl/list.hpp>
#include <boost/mpl/fold.hpp>
#include <boost/mpl/plus.hpp>
#include <boost/mpl/sizeof.hpp>
#include <boost/mpl/int.hpp>
#include <boost/mpl/placeholders.hpp>
#include <iostream>

namespace mpl = boost::mpl;
using namespace mpl::placeholders;

int main() {
    typedef mpl::list<char, short, int, long, double> types;

    // 计算所有类型大小之和（编译期）
    typedef mpl::fold<
        types,
        mpl::int_<0>,
        mpl::plus<_1, mpl::sizeof_<_2>>
    >::type total_size;

    std::cout << "所有类型大小总和: " << total_size::value << " 字节" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.MPL 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/mpl/doc/index.html)
- [MPL 参考手册](https://www.boost.org/doc/libs/1_90_0/libs/mpl/doc/refmanual.html)
- [MPL 教程](https://www.boost.org/doc/libs/1_90_0/libs/mpl/doc/tutorial.html)

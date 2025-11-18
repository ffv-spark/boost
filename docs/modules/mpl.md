# Boost.MPL - 元编程库

## 概述

Boost.MPL (Metaprogramming Library) 提供编译期元编程工具，支持类型列表、算法和元函数。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/at.hpp>
#include <iostream>
#include <typeinfo>

int main() {
    using namespace boost::mpl;
    
    // 创建类型序列
    typedef vector<int, double, char> types;
    
    // 获取元素
    typedef at_c<types, 0>::type first;   // int
    typedef at_c<types, 1>::type second;  // double
    
    std::cout << "第一个类型: " << typeid(first).name() << std::endl;
    std::cout << "第二个类型: " << typeid(second).name() << std::endl;
    
    return 0;
}
```

---

## 类型序列

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/list.hpp>
#include <boost/mpl/size.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    // vector
    typedef vector<int, double, char, float> vec;
    std::cout << "vector 大小: " << size<vec>::value << std::endl;
    
    // list
    typedef list<int, double, char> lst;
    std::cout << "list 大小: " << size<lst>::value << std::endl;
    
    return 0;
}
```

---

## 序列操作

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/push_back.hpp>
#include <boost/mpl/push_front.hpp>
#include <boost/mpl/pop_back.hpp>
#include <boost/mpl/size.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    typedef vector<int, double> vec1;
    
    // 添加到末尾
    typedef push_back<vec1, char>::type vec2;
    std::cout << "push_back 后大小: " << size<vec2>::value << std::endl;
    
    // 添加到开头
    typedef push_front<vec1, float>::type vec3;
    std::cout << "push_front 后大小: " << size<vec3>::value << std::endl;
    
    // 移除末尾
    typedef pop_back<vec2>::type vec4;
    std::cout << "pop_back 后大小: " << size<vec4>::value << std::endl;
    
    return 0;
}
```

---

## 算法：transform

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/transform.hpp>
#include <boost/mpl/at.hpp>
#include <iostream>
#include <typeinfo>

template <typename T>
struct add_pointer {
    typedef T* type;
};

int main() {
    using namespace boost::mpl;
    
    typedef vector<int, double, char> types;
    
    // 转换所有类型为指针
    typedef transform<types, add_pointer<_1>>::type pointer_types;
    
    typedef at_c<pointer_types, 0>::type first;  // int*
    typedef at_c<pointer_types, 1>::type second; // double*
    
    std::cout << "第一个: " << typeid(first).name() << std::endl;
    std::cout << "第二个: " << typeid(second).name() << std::endl;
    
    return 0;
}
```

---

## 算法：filter

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/copy_if.hpp>
#include <boost/mpl/back_inserter.hpp>
#include <boost/mpl/size.hpp>
#include <boost/type_traits.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    typedef vector<int, double, char*, float, int*> types;
    
    // 过滤出指针类型
    typedef copy_if<
        types,
        boost::is_pointer<_1>,
        back_inserter<vector<>>
    >::type pointer_types;
    
    std::cout << "原始大小: " << size<types>::value << std::endl;
    std::cout << "指针类型数量: " << size<pointer_types>::value << std::endl;
    
    return 0;
}
```

---

## 算法：find

```cpp
#include <boost/mpl/vector.hpp>
#include <boost/mpl/find.hpp>
#include <boost/mpl/distance.hpp>
#include <boost/mpl/begin.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    typedef vector<int, double, char, float> types;
    
    // 查找 char
    typedef find<types, char>::type iter;
    typedef begin<types>::type begin_iter;
    
    int index = distance<begin_iter, iter>::value;
    std::cout << "char 的索引: " << index << std::endl;
    
    return 0;
}
```

---

## 条件判断

```cpp
#include <boost/mpl/if.hpp>
#include <boost/mpl/bool.hpp>
#include <boost/type_traits.hpp>
#include <iostream>
#include <typeinfo>

template <typename T>
struct select_type {
    typedef typename boost::mpl::if_<
        boost::is_integral<T>,
        int,
        double
    >::type type;
};

int main() {
    typedef select_type<char>::type type1;    // int
    typedef select_type<float>::type type2;   // double
    
    std::cout << "char -> " << typeid(type1).name() << std::endl;
    std::cout << "float -> " << typeid(type2).name() << std::endl;
    
    return 0;
}
```

---

## 算术运算

```cpp
#include <boost/mpl/int.hpp>
#include <boost/mpl/plus.hpp>
#include <boost/mpl/minus.hpp>
#include <boost/mpl/multiplies.hpp>
#include <boost/mpl/divides.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    typedef int_<5> five;
    typedef int_<3> three;
    
    std::cout << "5 + 3 = " << plus<five, three>::value << std::endl;
    std::cout << "5 - 3 = " << minus<five, three>::value << std::endl;
    std::cout << "5 * 3 = " << multiplies<five, three>::value << std::endl;
    std::cout << "6 / 2 = " << divides<int_<6>, int_<2>>::value << std::endl;
    
    return 0;
}
```

---

## 逻辑运算

```cpp
#include <boost/mpl/bool.hpp>
#include <boost/mpl/and.hpp>
#include <boost/mpl/or.hpp>
#include <boost/mpl/not.hpp>
#include <iostream>

int main() {
    using namespace boost::mpl;
    
    typedef true_ t;
    typedef false_ f;
    
    std::cout << std::boolalpha;
    std::cout << "true && true = " << and_<t, t>::value << std::endl;
    std::cout << "true && false = " << and_<t, f>::value << std::endl;
    std::cout << "true || false = " << or_<t, f>::value << std::endl;
    std::cout << "!true = " << not_<t>::value << std::endl;
    
    return 0;
}
```

---

## 递归元编程

```cpp
#include <boost/mpl/int.hpp>
#include <boost/mpl/if.hpp>
#include <boost/mpl/multiplies.hpp>
#include <boost/mpl/minus.hpp>
#include <iostream>

template <int N>
struct factorial {
    typedef typename boost::mpl::if_c<
        N == 0,
        boost::mpl::int_<1>,
        boost::mpl::multiplies<
            boost::mpl::int_<N>,
            typename factorial<N-1>::type
        >
    >::type type;
    
    static const int value = type::value;
};

template <>
struct factorial<0> {
    typedef boost::mpl::int_<1> type;
    static const int value = 1;
};

int main() {
    std::cout << "5! = " << factorial<5>::value << std::endl;
    std::cout << "10! = " << factorial<10>::value << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.MPL 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/mpl/doc/index.html)
- [元编程教程](https://www.boost.org/doc/libs/1_90_0/libs/mpl/doc/tutorial/tutorial-metafunctions.html)

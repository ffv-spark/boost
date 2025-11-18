# Boost.Preprocessor - 预处理器元编程库

## 概述

Boost.Preprocessor 提供宏编程工具，支持循环、条件、列表操作等预处理器元编程。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/preprocessor/repetition/repeat.hpp>
#include <iostream>

#define PRINT_NUMBER(z, n, text) std::cout << n << " ";

int main() {
    // 重复执行宏10次
    BOOST_PP_REPEAT(10, PRINT_NUMBER, ~)
    std::cout << std::endl;
    
    return 0;
}
```

---

## 算术运算

```cpp
#include <boost/preprocessor/arithmetic.hpp>
#include <iostream>

#define A 5
#define B 3

int main() {
    // 加法
    std::cout << "5 + 3 = " << BOOST_PP_ADD(A, B) << std::endl;
    
    // 减法
    std::cout << "5 - 3 = " << BOOST_PP_SUB(A, B) << std::endl;
    
    // 乘法
    std::cout << "5 * 3 = " << BOOST_PP_MUL(A, B) << std::endl;
    
    // 除法
    std::cout << "6 / 2 = " << BOOST_PP_DIV(6, 2) << std::endl;
    
    // 取模
    std::cout << "7 % 3 = " << BOOST_PP_MOD(7, 3) << std::endl;
    
    return 0;
}
```

---

## 条件判断

```cpp
#include <boost/preprocessor/comparison.hpp>
#include <boost/preprocessor/control/if.hpp>
#include <iostream>

#define VALUE 10

int main() {
    // 相等
    std::cout << "10 == 10: " << BOOST_PP_EQUAL(VALUE, 10) << std::endl;
    
    // 不等
    std::cout << "10 != 5: " << BOOST_PP_NOT_EQUAL(VALUE, 5) << std::endl;
    
    // 大于
    std::cout << "10 > 5: " << BOOST_PP_GREATER(VALUE, 5) << std::endl;
    
    // 小于
    std::cout << "10 < 20: " << BOOST_PP_LESS(VALUE, 20) << std::endl;
    
    // 条件选择
    #define SELECT(cond) BOOST_PP_IF(cond, true, false)
    std::cout << "IF(1): " << SELECT(1) << std::endl;
    std::cout << "IF(0): " << SELECT(0) << std::endl;
    
    return 0;
}
```

---

## 循环重复

```cpp
#include <boost/preprocessor/repetition/repeat.hpp>
#include <iostream>

// 生成case语句
#define CASE_STATEMENT(z, n, text) \
    case n: std::cout << "Case " << n << std::endl; break;

void process(int value) {
    switch(value) {
        BOOST_PP_REPEAT(10, CASE_STATEMENT, ~)
        default: std::cout << "默认" << std::endl;
    }
}

int main() {
    process(3);
    process(7);
    process(15);
    
    return 0;
}
```

---

## 序列操作

```cpp
#include <boost/preprocessor/seq.hpp>
#include <iostream>

#define MY_SEQ (1)(2)(3)(4)(5)

// 遍历序列
#define PRINT_ELEM(r, data, elem) std::cout << elem << " ";

int main() {
    std::cout << "序列元素: ";
    BOOST_PP_SEQ_FOR_EACH(PRINT_ELEM, ~, MY_SEQ)
    std::cout << std::endl;
    
    // 序列大小
    std::cout << "序列大小: " << BOOST_PP_SEQ_SIZE(MY_SEQ) << std::endl;
    
    // 获取元素
    std::cout << "第一个元素: " << BOOST_PP_SEQ_HEAD(MY_SEQ) << std::endl;
    std::cout << "第三个元素: " << BOOST_PP_SEQ_ELEM(2, MY_SEQ) << std::endl;
    
    return 0;
}
```

---

## 列表操作

```cpp
#include <boost/preprocessor/list.hpp>
#include <iostream>

#define MY_LIST (1, (2, (3, (4, BOOST_PP_NIL))))

// 遍历列表
#define PRINT_LIST_ELEM(r, data, elem) std::cout << elem << " ";

int main() {
    std::cout << "列表元素: ";
    BOOST_PP_LIST_FOR_EACH(PRINT_LIST_ELEM, ~, MY_LIST)
    std::cout << std::endl;
    
    // 列表大小
    std::cout << "列表大小: " << BOOST_PP_LIST_SIZE(MY_LIST) << std::endl;
    
    // 获取元素
    std::cout << "第一个元素: " << BOOST_PP_LIST_FIRST(MY_LIST) << std::endl;
    
    return 0;
}
```

---

## 元组操作

```cpp
#include <boost/preprocessor/tuple.hpp>
#include <iostream>

#define MY_TUPLE (1, 2, 3, 4, 5)

int main() {
    // 元组大小
    std::cout << "元组大小: " << BOOST_PP_TUPLE_SIZE(MY_TUPLE) << std::endl;
    
    // 获取元素
    std::cout << "第一个元素: " << BOOST_PP_TUPLE_ELEM(0, MY_TUPLE) << std::endl;
    std::cout << "第三个元素: " << BOOST_PP_TUPLE_ELEM(2, MY_TUPLE) << std::endl;
    
    // 反转元组
    #define REVERSED BOOST_PP_TUPLE_REVERSE(MY_TUPLE)
    std::cout << "反转后第一个: " << BOOST_PP_TUPLE_ELEM(0, REVERSED) << std::endl;
    
    return 0;
}
```

---

## 字符串化

```cpp
#include <boost/preprocessor/stringize.hpp>
#include <iostream>

#define VALUE 42
#define NAME hello

int main() {
    // 转换为字符串
    std::cout << "VALUE as string: " << BOOST_PP_STRINGIZE(VALUE) << std::endl;
    std::cout << "NAME as string: " << BOOST_PP_STRINGIZE(NAME) << std::endl;
    
    // 实际应用
    #define DEFINE_GETTER(type, name) \
        type get_##name() const { return name##_; } \
        const char* get_##name##_name() const { return BOOST_PP_STRINGIZE(name); }
    
    struct Data {
        DEFINE_GETTER(int, value)
        DEFINE_GETTER(double, price)
    private:
        int value_ = 42;
        double price_ = 99.99;
    };
    
    Data d;
    std::cout << d.get_value_name() << ": " << d.get_value() << std::endl;
    std::cout << d.get_price_name() << ": " << d.get_price() << std::endl;
    
    return 0;
}
```

---

## 连接标识符

```cpp
#include <boost/preprocessor/cat.hpp>
#include <iostream>

#define PREFIX my_
#define SUFFIX _var

int main() {
    // 连接标识符
    int BOOST_PP_CAT(PREFIX, value) = 10;
    int BOOST_PP_CAT(num, SUFFIX) = 20;
    
    std::cout << "my_value: " << my_value << std::endl;
    std::cout << "num_var: " << num_var << std::endl;
    
    return 0;
}
```

---

## 生成代码

```cpp
#include <boost/preprocessor/repetition/enum.hpp>
#include <boost/preprocessor/repetition/enum_params.hpp>
#include <iostream>

// 生成参数列表
#define PARAM(z, n, text) int param##n

template<BOOST_PP_ENUM(5, PARAM, ~)>
struct MultiParam {
    void print() {
        std::cout << "参数数量: 5" << std::endl;
    }
};

int main() {
    MultiParam<1, 2, 3, 4, 5> mp;
    mp.print();
    
    return 0;
}
```

---

## 生成结构体

```cpp
#include <boost/preprocessor/seq.hpp>
#include <iostream>
#include <string>

#define FIELDS (int, id)(std::string, name)(double, salary)

// 生成成员变量
#define DECLARE_FIELD(r, data, elem) \
    BOOST_PP_TUPLE_ELEM(0, elem) BOOST_PP_TUPLE_ELEM(1, elem);

// 生成getter
#define DECLARE_GETTER(r, data, elem) \
    BOOST_PP_TUPLE_ELEM(0, elem) get_##BOOST_PP_TUPLE_ELEM(1, elem)() const { \
        return BOOST_PP_TUPLE_ELEM(1, elem); \
    }

struct Employee {
    BOOST_PP_SEQ_FOR_EACH(DECLARE_FIELD, ~, FIELDS)
    BOOST_PP_SEQ_FOR_EACH(DECLARE_GETTER, ~, FIELDS)
};

int main() {
    Employee e;
    e.id = 1001;
    e.name = "Alice";
    e.salary = 75000.0;
    
    std::cout << "ID: " << e.get_id() << std::endl;
    std::cout << "Name: " << e.get_name() << std::endl;
    std::cout << "Salary: " << e.get_salary() << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Preprocessor 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/preprocessor/doc/index.html)
- [预处理器元编程指南](https://www.boost.org/doc/libs/1_90_0/libs/preprocessor/doc/ref.html)

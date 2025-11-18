# Boost.Preprocessor - 预处理器元编程库

## 概述

Boost.Preprocessor 提供强大的预处理器宏，用于代码生成和预处理期间的元编程。

**类型**: 仅头文件库

---

## 快速开始 - 重复

```cpp
#include <boost/preprocessor/repetition/repeat.hpp>
#include <iostream>

// 生成打印语句的宏
#define PRINT_NUMBER(z, n, text) \
    std::cout << text << n << std::endl;

int main() {
    // 重复10次
    BOOST_PP_REPEAT(10, PRINT_NUMBER, "数字: ")

    return 0;
}
```

---

## 序列操作

```cpp
#include <boost/preprocessor/seq.hpp>
#include <iostream>

// 定义序列（注意：没有逗号）
#define MY_SEQ (int)(double)(char)(float)

// 序列大小
#define SEQ_SIZE BOOST_PP_SEQ_SIZE(MY_SEQ)

// 访问元素
#define FIRST_ELEM BOOST_PP_SEQ_ELEM(0, MY_SEQ)  // int
#define SECOND_ELEM BOOST_PP_SEQ_ELEM(1, MY_SEQ) // double

// 遍历序列
#define PRINT_TYPE(r, data, elem) \
    std::cout << "类型大小: " << sizeof(elem) << std::endl;

int main() {
    std::cout << "序列大小: " << SEQ_SIZE << std::endl;

    BOOST_PP_SEQ_FOR_EACH(PRINT_TYPE, ~, MY_SEQ)

    return 0;
}
```

---

## 列表操作

```cpp
#include <boost/preprocessor/list.hpp>
#include <iostream>

// 定义列表（使用括号和 BOOST_PP_NIL）
#define MY_LIST (int, (double, (char, BOOST_PP_NIL)))

// 列表大小
#define LIST_SIZE BOOST_PP_LIST_SIZE(MY_LIST)

// 访问元素
#define FIRST BOOST_PP_LIST_FIRST(MY_LIST)           // int
#define REST BOOST_PP_LIST_REST(MY_LIST)             // (double, (char, NIL))
#define SECOND BOOST_PP_LIST_FIRST(REST)             // double

// 遍历列表
#define PRINT_SIZE(r, data, elem) \
    std::cout << #elem << " 大小: " << sizeof(elem) << std::endl;

int main() {
    std::cout << "列表大小: " << LIST_SIZE << std::endl;

    BOOST_PP_LIST_FOR_EACH(PRINT_SIZE, ~, MY_LIST)

    return 0;
}
```

---

## 元组操作

```cpp
#include <boost/preprocessor/tuple.hpp>
#include <iostream>

// 定义元组
#define MY_TUPLE (int, double, char)

// 元组大小
#define TUPLE_SIZE BOOST_PP_TUPLE_SIZE(MY_TUPLE)

// 访问元素
#define ELEM_0 BOOST_PP_TUPLE_ELEM(0, MY_TUPLE)  // int
#define ELEM_1 BOOST_PP_TUPLE_ELEM(1, MY_TUPLE)  // double
#define ELEM_2 BOOST_PP_TUPLE_ELEM(2, MY_TUPLE)  // char

int main() {
    std::cout << "元组大小: " << TUPLE_SIZE << std::endl;

    ELEM_0 i = 42;
    ELEM_1 d = 3.14;
    ELEM_2 c = 'A';

    std::cout << "i = " << i << std::endl;
    std::cout << "d = " << d << std::endl;
    std::cout << "c = " << c << std::endl;

    return 0;
}
```

---

## 算术运算

```cpp
#include <boost/preprocessor/arithmetic.hpp>
#include <iostream>

#define A 10
#define B 5

// 加法
#define SUM BOOST_PP_ADD(A, B)          // 15

// 减法
#define DIFF BOOST_PP_SUB(A, B)         // 5

// 乘法
#define PRODUCT BOOST_PP_MUL(A, B)      // 50

// 除法
#define QUOTIENT BOOST_PP_DIV(A, B)     // 2

// 取模
#define REMAINDER BOOST_PP_MOD(A, B)    // 0

int main() {
    std::cout << A << " + " << B << " = " << SUM << std::endl;
    std::cout << A << " - " << B << " = " << DIFF << std::endl;
    std::cout << A << " * " << B << " = " << PRODUCT << std::endl;
    std::cout << A << " / " << B << " = " << QUOTIENT << std::endl;
    std::cout << A << " % " << B << " = " << REMAINDER << std::endl;

    return 0;
}
```

---

## 比较运算

```cpp
#include <boost/preprocessor/comparison.hpp>
#include <boost/preprocessor/control/if.hpp>
#include <iostream>

#define X 10
#define Y 20

// 等于
#define IS_EQUAL BOOST_PP_EQUAL(X, Y)       // 0 (false)

// 不等于
#define NOT_EQUAL BOOST_PP_NOT_EQUAL(X, Y)  // 1 (true)

// 小于
#define LESS BOOST_PP_LESS(X, Y)            // 1 (true)

// 大于
#define GREATER BOOST_PP_GREATER(X, Y)      // 0 (false)

// 条件选择
#define RESULT BOOST_PP_IF(LESS, "X < Y", "X >= Y")

int main() {
    std::cout << "X == Y: " << IS_EQUAL << std::endl;
    std::cout << "X != Y: " << NOT_EQUAL << std::endl;
    std::cout << "X < Y: " << LESS << std::endl;
    std::cout << "X > Y: " << GREATER << std::endl;
    std::cout << "结果: " << RESULT << std::endl;

    return 0;
}
```

---

## 字符串化和连接

```cpp
#include <boost/preprocessor/stringize.hpp>
#include <boost/preprocessor/cat.hpp>
#include <iostream>

#define MY_VAR 42

// 字符串化
#define VAR_NAME BOOST_PP_STRINGIZE(MY_VAR)  // "MY_VAR"

// 连接标识符
#define MAKE_VAR(prefix, suffix) BOOST_PP_CAT(prefix, suffix)

int MAKE_VAR(my_, variable) = 100;  // 生成 my_variable

int main() {
    std::cout << VAR_NAME << " = " << MY_VAR << std::endl;
    std::cout << "my_variable = " << my_variable << std::endl;

    return 0;
}
```

---

## 自动生成代码

```cpp
#include <boost/preprocessor/repetition/enum.hpp>
#include <boost/preprocessor/repetition/enum_params.hpp>
#include <iostream>

// 生成参数列表
#define GEN_PARAM(z, n, text) text##n

// 生成打印函数
template<typename T>
void print_values(BOOST_PP_ENUM_PARAMS(5, T arg)) {
    std::cout << "Values: ";
    BOOST_PP_REPEAT(5, PRINT_ARG, arg)
    std::cout << std::endl;
}

#define PRINT_ARG(z, n, prefix) \
    std::cout << prefix##n << " ";

int main() {
    print_values(1, 2, 3, 4, 5);

    return 0;
}
```

---

## 枚举生成

```cpp
#include <boost/preprocessor/seq/for_each.hpp>
#include <boost/preprocessor/stringize.hpp>
#include <iostream>

// 定义枚举值序列
#define COLOR_SEQ (RED)(GREEN)(BLUE)(YELLOW)

// 生成枚举
#define ENUM_ENTRY(r, data, elem) elem,

enum Color {
    BOOST_PP_SEQ_FOR_EACH(ENUM_ENTRY, ~, COLOR_SEQ)
    COLOR_COUNT
};

// 生成字符串转换函数
#define CASE_ENTRY(r, data, elem) \
    case elem: return BOOST_PP_STRINGIZE(elem);

const char* color_to_string(Color c) {
    switch (c) {
        BOOST_PP_SEQ_FOR_EACH(CASE_ENTRY, ~, COLOR_SEQ)
        default: return "UNKNOWN";
    }
}

int main() {
    Color c = GREEN;
    std::cout << "颜色: " << color_to_string(c) << std::endl;
    std::cout << "颜色总数: " << COLOR_COUNT << std::endl;

    return 0;
}
```

---

## 结构体成员生成

```cpp
#include <boost/preprocessor/seq/for_each.hpp>
#include <boost/preprocessor/tuple/elem.hpp>
#include <iostream>

// 定义成员：(类型, 名称)
#define MEMBERS \
    ((int, id)) \
    ((std::string, name)) \
    ((double, salary))

// 生成成员变量
#define DECLARE_MEMBER(r, data, elem) \
    BOOST_PP_TUPLE_ELEM(0, elem) BOOST_PP_TUPLE_ELEM(1, elem);

// 生成 getter
#define DECLARE_GETTER(r, data, elem) \
    BOOST_PP_TUPLE_ELEM(0, elem) get_##BOOST_PP_TUPLE_ELEM(1, elem)() const { \
        return BOOST_PP_TUPLE_ELEM(1, elem); \
    }

class Employee {
public:
    BOOST_PP_SEQ_FOR_EACH(DECLARE_MEMBER, ~, MEMBERS)
    BOOST_PP_SEQ_FOR_EACH(DECLARE_GETTER, ~, MEMBERS)
};

int main() {
    Employee emp;
    emp.id = 123;
    emp.name = "张三";
    emp.salary = 5000.0;

    std::cout << "ID: " << emp.get_id() << std::endl;
    std::cout << "姓名: " << emp.get_name() << std::endl;
    std::cout << "工资: " << emp.get_salary() << std::endl;

    return 0;
}
```

---

## 可变参数宏

```cpp
#include <boost/preprocessor/variadic.hpp>
#include <iostream>

// 获取可变参数数量
#define COUNT_ARGS(...) BOOST_PP_VARIADIC_SIZE(__VA_ARGS__)

// 转换为序列
#define ARGS_TO_SEQ(...) BOOST_PP_VARIADIC_TO_SEQ(__VA_ARGS__)

// 打印可变参数
#define PRINT_ARGS(...) \
    std::cout << "参数数量: " << COUNT_ARGS(__VA_ARGS__) << std::endl;

int main() {
    PRINT_ARGS(1, 2, 3);
    PRINT_ARGS(a, b, c, d, e);

    return 0;
}
```

---

## 循环展开

```cpp
#include <boost/preprocessor/repetition/repeat.hpp>
#include <iostream>

// 手动循环展开以提高性能
#define UNROLL_SUM(z, n, array) sum += array[n];

void optimized_sum(const int* array, int size) {
    int sum = 0;

    // 展开前8次迭代
    BOOST_PP_REPEAT(8, UNROLL_SUM, array)

    // 处理剩余部分
    for (int i = 8; i < size; ++i) {
        sum += array[i];
    }

    std::cout << "总和: " << sum << std::endl;
}

int main() {
    int array[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    optimized_sum(array, 10);

    return 0;
}
```

---

## 参考资源

- [Boost.Preprocessor 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/preprocessor/doc/index.html)
- [预处理器参考](https://www.boost.org/doc/libs/1_90_0/libs/preprocessor/doc/ref.html)

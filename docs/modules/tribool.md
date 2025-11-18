# Boost.Tribool - 三态布尔库

## 概述

Boost.Tribool 提供三态布尔类型，支持 true、false 和 indeterminate（不确定）三种状态。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

int main() {
    tribool t = true;
    tribool f = false;
    tribool u = indeterminate;

    std::cout << "true: " << (t ? "是" : "否") << std::endl;
    std::cout << "false: " << (f ? "是" : "否") << std::endl;
    std::cout << "indeterminate 是否确定: "
              << (indeterminate(u) ? "不确定" : "确定") << std::endl;

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/logic/tribool.hpp>
#include <boost/logic/tribool_io.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

int main() {
    tribool t = true;
    tribool f = false;
    tribool u = indeterminate;

    // 输出（需要包含 tribool_io.hpp）
    std::cout << "true: " << t << std::endl;
    std::cout << "false: " << f << std::endl;
    std::cout << "indeterminate: " << u << std::endl;

    // 检查状态
    if (t) {
        std::cout << "t 为真" << std::endl;
    }

    if (!f) {
        std::cout << "f 为假" << std::endl;
    }

    if (indeterminate(u)) {
        std::cout << "u 不确定" << std::endl;
    }

    return 0;
}
```

---

## 逻辑运算

```cpp
#include <boost/logic/tribool.hpp>
#include <boost/logic/tribool_io.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

void print_result(const std::string& expr, tribool result) {
    std::cout << expr << " = ";
    if (result) {
        std::cout << "true";
    } else if (!result) {
        std::cout << "false";
    } else {
        std::cout << "indeterminate";
    }
    std::cout << std::endl;
}

int main() {
    tribool t = true;
    tribool f = false;
    tribool u = indeterminate;

    // AND 运算
    print_result("true && true", t && t);
    print_result("true && false", t && f);
    print_result("true && indeterminate", t && u);
    print_result("false && indeterminate", f && u);

    std::cout << std::endl;

    // OR 运算
    print_result("true || false", t || f);
    print_result("true || indeterminate", t || u);
    print_result("false || false", f || f);
    print_result("false || indeterminate", f || u);

    std::cout << std::endl;

    // NOT 运算
    print_result("!true", !t);
    print_result("!false", !f);
    print_result("!indeterminate", !u);

    return 0;
}
```

---

## 三态逻辑表

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

void print_tribool(tribool b) {
    if (b) std::cout << "T";
    else if (!b) std::cout << "F";
    else std::cout << "?";
}

void print_truth_table() {
    tribool values[] = {true, false, indeterminate};
    const char* labels[] = {"T", "F", "?"};

    std::cout << "AND 真值表:\n";
    std::cout << "    T F ?\n";
    for (int i = 0; i < 3; ++i) {
        std::cout << labels[i] << " | ";
        for (int j = 0; j < 3; ++j) {
            print_tribool(values[i] && values[j]);
            std::cout << " ";
        }
        std::cout << std::endl;
    }

    std::cout << "\nOR 真值表:\n";
    std::cout << "    T F ?\n";
    for (int i = 0; i < 3; ++i) {
        std::cout << labels[i] << " | ";
        for (int j = 0; j < 3; ++j) {
            print_tribool(values[i] || values[j]);
            std::cout << " ";
        }
        std::cout << std::endl;
    }
}

int main() {
    print_truth_table();
    return 0;
}
```

---

## 数据库查询示例

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>
#include <string>
#include <vector>

using boost::logic::tribool;
using boost::logic::indeterminate;

struct Person {
    std::string name;
    tribool is_student;  // 可能未知
    tribool has_job;     // 可能未知

    Person(const std::string& n, tribool s, tribool j)
        : name(n), is_student(s), has_job(j) {}
};

int main() {
    std::vector<Person> people = {
        {"Alice", true, false},
        {"Bob", false, true},
        {"Charlie", indeterminate, true},  // 不确定是否学生
        {"David", false, indeterminate}    // 不确定是否有工作
    };

    std::cout << "学生且无工作的人:\n";
    for (const auto& p : people) {
        tribool result = p.is_student && !p.has_job;

        if (result) {
            std::cout << "  确定: " << p.name << std::endl;
        } else if (indeterminate(result)) {
            std::cout << "  可能: " << p.name << std::endl;
        }
    }

    return 0;
}
```

---

## 条件判断

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

std::string check_access(tribool has_permission) {
    if (has_permission) {
        return "允许访问";
    } else if (!has_permission) {
        return "拒绝访问";
    } else if (indeterminate(has_permission)) {
        return "权限未确定，需要进一步验证";
    }
    return "未知状态";
}

int main() {
    std::cout << check_access(true) << std::endl;
    std::cout << check_access(false) << std::endl;
    std::cout << check_access(indeterminate) << std::endl;

    return 0;
}
```

---

## 与普通 bool 的转换

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

int main() {
    // bool 到 tribool 的隐式转换
    tribool t = true;
    tribool f = false;

    std::cout << "从 bool 转换: " << (t ? "真" : "假或不确定") << std::endl;

    // tribool 到 bool 需要显式处理
    tribool u = indeterminate;

    // 错误的做法（会编译但可能有意外结果）
    // bool b = u;  // 不推荐

    // 正确的做法
    if (indeterminate(u)) {
        std::cout << "值不确定，无法转换为 bool" << std::endl;
    } else {
        bool b = (u == true);
        std::cout << "转换为 bool: " << b << std::endl;
    }

    return 0;
}
```

---

## 安全的布尔检查

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

class SafeChecker {
public:
    static bool is_definitely_true(tribool value) {
        return value == true;
    }

    static bool is_definitely_false(tribool value) {
        return value == false;
    }

    static bool is_unknown(tribool value) {
        return indeterminate(value);
    }

    static bool is_true_or_unknown(tribool value) {
        return value || indeterminate(value);
    }
};

int main() {
    tribool values[] = {true, false, indeterminate};
    const char* labels[] = {"true", "false", "indeterminate"};

    for (int i = 0; i < 3; ++i) {
        std::cout << "\n检查 " << labels[i] << ":\n";
        std::cout << "  确定为真: " << SafeChecker::is_definitely_true(values[i]) << std::endl;
        std::cout << "  确定为假: " << SafeChecker::is_definitely_false(values[i]) << std::endl;
        std::cout << "  不确定: " << SafeChecker::is_unknown(values[i]) << std::endl;
        std::cout << "  真或不确定: " << SafeChecker::is_true_or_unknown(values[i]) << std::endl;
    }

    return 0;
}
```

---

## 比较运算

```cpp
#include <boost/logic/tribool.hpp>
#include <boost/logic/tribool_io.hpp>
#include <iostream>

using boost::logic::tribool;
using boost::logic::indeterminate;

int main() {
    tribool t = true;
    tribool f = false;
    tribool u = indeterminate;

    // 相等比较
    std::cout << "true == true: " << (t == t) << std::endl;
    std::cout << "true == false: " << (t == f) << std::endl;
    std::cout << "true == indeterminate: " << (t == u) << std::endl;
    std::cout << "indeterminate == indeterminate: " << (u == u) << std::endl;

    std::cout << std::endl;

    // 不等比较
    std::cout << "true != false: " << (t != f) << std::endl;
    std::cout << "true != indeterminate: " << (t != u) << std::endl;

    return 0;
}
```

---

## SQL 空值语义

```cpp
#include <boost/logic/tribool.hpp>
#include <iostream>
#include <string>

using boost::logic::tribool;
using boost::logic::indeterminate;

struct SQLValue {
    std::string data;
    bool is_null;

    SQLValue(const std::string& d) : data(d), is_null(false) {}
    SQLValue() : is_null(true) {}

    tribool equals(const SQLValue& other) const {
        if (is_null || other.is_null) {
            return indeterminate;  // NULL 比较返回不确定
        }
        return data == other.data;
    }
};

int main() {
    SQLValue val1("hello");
    SQLValue val2("hello");
    SQLValue val3("world");
    SQLValue null_val;

    std::cout << "hello == hello: ";
    if (val1.equals(val2)) std::cout << "true";
    else if (!val1.equals(val2)) std::cout << "false";
    else std::cout << "indeterminate";
    std::cout << std::endl;

    std::cout << "hello == NULL: ";
    if (val1.equals(null_val)) std::cout << "true";
    else if (!val1.equals(null_val)) std::cout << "false";
    else std::cout << "indeterminate";
    std::cout << std::endl;

    std::cout << "NULL == NULL: ";
    if (null_val.equals(null_val)) std::cout << "true";
    else if (!null_val.equals(null_val)) std::cout << "false";
    else std::cout << "indeterminate";
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Tribool 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/tribool.html)
- [三值逻辑](https://en.wikipedia.org/wiki/Three-valued_logic)

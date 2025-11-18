# Boost.Stacktrace - 栈追踪库

## 概述

Boost.Stacktrace 提供捕获和打印函数调用栈的功能，用于调试和错误报告。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/stacktrace.hpp>
#include <iostream>

void function_c() {
    std::cout << "当前调用栈:\\n";
    std::cout << boost::stacktrace::stacktrace() << std::endl;
}

void function_b() {
    function_c();
}

void function_a() {
    function_b();
}

int main() {
    function_a();
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_stacktrace_basic -ldl -DBOOST_STACKTRACE_USE_BACKTRACE`

---

## 异常中的栈追踪

```cpp
#include <boost/stacktrace.hpp>
#include <boost/exception/all.hpp>
#include <iostream>
#include <exception>

typedef boost::error_info<struct tag_stacktrace, boost::stacktrace::stacktrace> traced;

template <class E>
void throw_with_trace(const E& e) {
    throw boost::enable_error_info(e) << traced(boost::stacktrace::stacktrace());
}

void problematic_function() {
    throw_with_trace(std::runtime_error("发生错误！"));
}

int main() {
    try {
        problematic_function();
    }
    catch (const boost::exception& e) {
        std::cerr << "异常: " << boost::diagnostic_information(e) << std::endl;
        
        if (const boost::stacktrace::stacktrace* st = boost::get_error_info<traced>(e)) {
            std::cerr << "调用栈:\\n" << *st << std::endl;
        }
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Stacktrace 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/stacktrace.html)

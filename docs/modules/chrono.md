# Boost.Chrono - 时间库

## 概述

Boost.Chrono 提供时间点、持续时间和时钟的操作，是 C++11 std::chrono 的前身。

**类型**: 需要编译的库

**注意**: C++11 引入了 std::chrono，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    // 创建时间点
    auto start = high_resolution_clock::now();

    // 模拟工作
    for (int i = 0; i < 1000000; ++i);

    auto end = high_resolution_clock::now();

    // 计算持续时间
    auto duration = end - start;

    std::cout << "耗时: " << duration << std::endl;
    std::cout << "毫秒: " << duration_cast<milliseconds>(duration) << std::endl;

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_chrono -lboost_system`

---

## 持续时间

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    // 创建不同单位的持续时间
    seconds sec(5);
    milliseconds ms(5000);
    microseconds us(5000000);

    std::cout << sec << std::endl;
    std::cout << ms << std::endl;
    std::cout << us << std::endl;

    // 转换
    auto ms_from_sec = duration_cast<milliseconds>(sec);
    std::cout << "5秒 = " << ms_from_sec.count() << " 毫秒" << std::endl;

    return 0;
}
```

---

## 时钟类型

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    // 系统时钟
    auto sys_now = system_clock::now();
    std::cout << "系统时间: " << sys_now << std::endl;

    // 稳定时钟（单调）
    auto steady_start = steady_clock::now();
    for (int i = 0; i < 1000000; ++i);
    auto steady_end = steady_clock::now();
    std::cout << "稳定时钟差: " << steady_end - steady_start << std::endl;

    // 高精度时钟
    auto hr_now = high_resolution_clock::now();
    std::cout << "高精度时间: " << hr_now << std::endl;

    return 0;
}
```

---

## 时间算术

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    hours h(2);
    minutes m(30);
    seconds s(45);

    // 加法
    auto total = h + m + s;
    std::cout << "总计: " << total << std::endl;
    std::cout << "秒数: " << duration_cast<seconds>(total) << std::endl;

    // 减法
    auto diff = h - m;
    std::cout << "差值: " << duration_cast<minutes>(diff) << std::endl;

    // 乘法
    auto doubled = s * 2;
    std::cout << "翻倍: " << doubled << std::endl;

    // 除法
    auto half = s / 2;
    std::cout << "减半: " << half << std::endl;

    return 0;
}
```

---

## 时间点操作

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    auto now = system_clock::now();
    std::cout << "当前: " << now << std::endl;

    // 添加时间
    auto future = now + hours(24);
    std::cout << "24小时后: " << future << std::endl;

    // 减去时间
    auto past = now - hours(12);
    std::cout << "12小时前: " << past << std::endl;

    // 时间点差值
    auto diff = future - past;
    std::cout << "差值: " << duration_cast<hours>(diff) << std::endl;

    return 0;
}
```

---

## 计时器

```cpp
#include <boost/chrono.hpp>
#include <iostream>
#include <vector>

using namespace boost::chrono;

void benchmark_operation() {
    auto start = high_resolution_clock::now();

    // 执行操作
    std::vector<int> v(1000000);
    for (size_t i = 0; i < v.size(); ++i) {
        v[i] = i * i;
    }

    auto end = high_resolution_clock::now();
    auto duration = end - start;

    std::cout << "操作耗时: ";
    std::cout << duration_cast<milliseconds>(duration) << std::endl;
}

int main() {
    benchmark_operation();
    return 0;
}
```

---

## CPU 时间

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    auto start = process_real_cpu_clock::now();

    // 执行 CPU 密集型任务
    double result = 0;
    for (int i = 0; i < 10000000; ++i) {
        result += i * 0.0001;
    }

    auto end = process_real_cpu_clock::now();
    auto cpu_time = end - start;

    std::cout << "CPU 时间: ";
    std::cout << duration_cast<milliseconds>(cpu_time) << std::endl;
    std::cout << "结果: " << result << std::endl;

    return 0;
}
```

---

## 时间格式化

```cpp
#include <boost/chrono.hpp>
#include <boost/chrono/chrono_io.hpp>
#include <iostream>
#include <locale>

int main() {
    using namespace boost::chrono;

    milliseconds ms(12345);

    // 默认格式
    std::cout << ms << std::endl;

    // 使用 IO 格式化
    std::cout << duration_fmt(duration_style::prefix) << ms << std::endl;
    std::cout << duration_fmt(duration_style::symbol) << ms << std::endl;

    return 0;
}
```

---

## 等待超时

```cpp
#include <boost/chrono.hpp>
#include <boost/thread.hpp>
#include <iostream>

using namespace boost::chrono;

int main() {
    auto start = steady_clock::now();

    // 等待2秒
    boost::this_thread::sleep_for(seconds(2));

    auto end = steady_clock::now();
    auto elapsed = end - start;

    std::cout << "实际等待: ";
    std::cout << duration_cast<milliseconds>(elapsed) << std::endl;

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_chrono -lboost_thread -lboost_system -lpthread`

---

## 周期性任务

```cpp
#include <boost/chrono.hpp>
#include <iostream>

using namespace boost::chrono;

int main() {
    auto next_run = steady_clock::now();
    milliseconds interval(500);  // 500ms 间隔

    for (int i = 0; i < 5; ++i) {
        // 计算下次运行时间
        next_run += interval;

        // 执行任务
        std::cout << "任务 " << i + 1 << " 执行" << std::endl;

        // 等待到下次运行时间
        auto now = steady_clock::now();
        if (next_run > now) {
            auto sleep_time = next_run - now;
            std::cout << "  等待 " << duration_cast<milliseconds>(sleep_time) 
                      << std::endl;
        }
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Chrono 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/chrono.html)
- [C++11 std::chrono](https://en.cppreference.com/w/cpp/chrono)

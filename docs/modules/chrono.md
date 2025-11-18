# Boost.Chrono - 时间点和时长库

## 概述

Boost.Chrono 提供时间点、时长和时钟功能。

**类型**: 需要编译链接的库

**链接库**: `-lboost_chrono -lboost_system`

**注意**: C++11 已引入 `<chrono>`

---

## 快速开始

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    // 时间点
    auto start = boost::chrono::high_resolution_clock::now();

    // 执行一些操作
    for (int i = 0; i < 1000000; ++i) {
        volatile int x = i * i;
    }

    auto end = boost::chrono::high_resolution_clock::now();

    // 计算时长
    auto duration = end - start;

    std::cout << "耗时: "
              << boost::chrono::duration_cast<boost::chrono::milliseconds>(duration).count()
              << " ms" << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_chrono -lboost_system -o example
```

---

## 时钟类型

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    // 系统时钟
    auto sys_now = boost::chrono::system_clock::now();

    // 高精度时钟
    auto hr_now = boost::chrono::high_resolution_clock::now();

    // 稳定时钟（单调递增）
    auto steady_now = boost::chrono::steady_clock::now();

    std::cout << "System clock: " << sys_now.time_since_epoch().count() << std::endl;

    return 0;
}
```

---

## 时长操作

```cpp
#include <boost/chrono.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    // 创建时长
    seconds sec(10);
    milliseconds ms(5000);
    microseconds us(1000000);

    // 转换
    auto sec_from_ms = duration_cast<seconds>(ms);
    std::cout << "5000 ms = " << sec_from_ms.count() << " s" << std::endl;

    // 算术运算
    seconds total = sec + sec_from_ms;
    std::cout << "Total: " << total.count() << " s" << std::endl;

    // 比较
    if (sec < ms) {
        std::cout << "10s < 5000ms" << std::endl;
    }

    return 0;
}
```

---

## 计时器

```cpp
#include <boost/chrono.hpp>
#include <boost/chrono/process_cpu_clocks.hpp>
#include <iostream>

void expensive_operation() {
    for (long i = 0; i < 10000000; ++i) {
        volatile double x = i * 1.5;
    }
}

int main() {
    boost::chrono::process_cpu_clock::time_point start =
        boost::chrono::process_cpu_clock::now();

    expensive_operation();

    boost::chrono::process_cpu_clock::time_point end =
        boost::chrono::process_cpu_clock::now();

    boost::chrono::process_cpu_clock::duration elapsed = end - start;

    std::cout << "CPU time: "
              << boost::chrono::duration_cast<boost::chrono::milliseconds>(elapsed).count()
              << " ms" << std::endl;

    return 0;
}
```

---

## I/O 格式化

```cpp
#include <boost/chrono.hpp>
#include <boost/chrono/chrono_io.hpp>
#include <iostream>

int main() {
    using namespace boost::chrono;

    seconds sec(65);
    std::cout << sec << std::endl;

    milliseconds ms(1234);
    std::cout << ms << std::endl;

    hours h(2);
    minutes m(30);
    auto total = h + m;
    std::cout << duration_cast<minutes>(total) << std::endl;

    return 0;
}
```

---

## 与 std::chrono 对比

```cpp
#include <boost/chrono.hpp>
#include <chrono>
#include <iostream>

int main() {
    // Boost.Chrono
    auto b_start = boost::chrono::high_resolution_clock::now();
    auto b_duration = boost::chrono::milliseconds(1000);

    // std::chrono (C++11)
    auto s_start = std::chrono::high_resolution_clock::now();
    auto s_duration = std::chrono::milliseconds(1000);

    // API 基本相同
    std::cout << "Boost: " << b_duration.count() << std::endl;
    std::cout << "Std: " << s_duration.count() << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Chrono 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/chrono.html)

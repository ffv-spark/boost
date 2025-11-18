# Boost.Timer - 计时器库

## 概述

Boost.Timer 提供简单易用的计时功能，用于性能测量和时间监控。

**类型**: 需要编译的库（某些组件）

---

## 快速开始

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::timer::auto_cpu_timer timer;

    // 模拟一些工作
    std::this_thread::sleep_for(std::chrono::seconds(1));

    // timer 析构时自动输出时间

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_timer -lboost_system`

---

## CPU 计时器

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <vector>

void expensive_operation() {
    std::vector<int> v(10000000);
    for (size_t i = 0; i < v.size(); ++i) {
        v[i] = i * i;
    }
}

int main() {
    boost::timer::cpu_timer timer;

    expensive_operation();

    timer.stop();

    std::cout << "操作耗时:\n" << timer.format() << std::endl;

    return 0;
}
```

---

## 手动计时

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::timer::cpu_timer timer;

    // 第一阶段
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    std::cout << "第一阶段:\n" << timer.format() << std::endl;

    // 第二阶段
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    timer.stop();
    std::cout << "总计:\n" << timer.format() << std::endl;

    return 0;
}
```

---

## 性能测试

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

void test_sort(size_t size) {
    std::vector<int> v(size);
    for (size_t i = 0; i < size; ++i) {
        v[i] = size - i;
    }

    boost::timer::cpu_timer timer;
    std::sort(v.begin(), v.end());
    timer.stop();

    std::cout << "排序 " << size << " 个元素:\n";
    std::cout << timer.format() << std::endl;
}

int main() {
    test_sort(100000);
    test_sort(1000000);
    test_sort(10000000);

    return 0;
}
```

---

## 暂停和恢复

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::timer::cpu_timer timer;

    // 工作1
    std::this_thread::sleep_for(std::chrono::milliseconds(200));

    // 暂停
    timer.stop();
    std::cout << "暂停时:\n" << timer.format() << std::endl;

    // 做一些不计时的工作
    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    // 恢复
    timer.resume();

    // 工作2
    std::this_thread::sleep_for(std::chrono::milliseconds(200));

    timer.stop();
    std::cout << "恢复后:\n" << timer.format() << std::endl;

    return 0;
}
```

---

## 获取时间值

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::timer::cpu_timer timer;

    std::this_thread::sleep_for(std::chrono::seconds(1));

    timer.stop();

    boost::timer::cpu_times times = timer.elapsed();

    // 纳秒
    std::cout << "墙上时间: " << times.wall / 1e9 << " 秒" << std::endl;
    std::cout << "用户时间: " << times.user / 1e9 << " 秒" << std::endl;
    std::cout << "系统时间: " << times.system / 1e9 << " 秒" << std::endl;

    return 0;
}
```

---

## 自定义输出格式

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    boost::timer::cpu_timer timer;

    std::this_thread::sleep_for(std::chrono::milliseconds(500));

    timer.stop();

    // 自定义格式
    // %w - 墙上时间
    // %u - 用户时间
    // %s - 系统时间
    // %t - 总CPU时间（用户+系统）
    std::cout << timer.format(6, "耗时: %w 秒\n") << std::endl;

    return 0;
}
```

---

## 算法对比

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

template<typename Func>
void benchmark(const std::string& name, Func func) {
    boost::timer::cpu_timer timer;
    func();
    timer.stop();

    std::cout << name << ":\n" << timer.format() << std::endl;
}

int main() {
    const int size = 1000000;
    std::vector<int> data(size);

    // 初始化数据
    for (int i = 0; i < size; ++i) {
        data[i] = size - i;
    }

    // 测试排序
    auto data_copy = data;
    benchmark("std::sort", [&]() {
        std::sort(data_copy.begin(), data_copy.end());
    });

    // 测试 stable_sort
    data_copy = data;
    benchmark("std::stable_sort", [&]() {
        std::stable_sort(data_copy.begin(), data_copy.end());
    });

    return 0;
}
```

---

## 进度监控

```cpp
#include <boost/timer/timer.hpp>
#include <iostream>
#include <thread>
#include <chrono>

void process_items(int total) {
    boost::timer::cpu_timer timer;

    for (int i = 1; i <= total; ++i) {
        // 模拟处理
        std::this_thread::sleep_for(std::chrono::milliseconds(100));

        if (i % 10 == 0) {
            timer.stop();
            auto elapsed = timer.elapsed();
            double seconds = elapsed.wall / 1e9;
            double items_per_sec = i / seconds;
            double remaining = (total - i) / items_per_sec;

            std::cout << "进度: " << i << "/" << total 
                      << ", 速度: " << items_per_sec << " 项/秒"
                      << ", 预计剩余: " << remaining << " 秒" << std::endl;

            timer.resume();
        }
    }
}

int main() {
    process_items(50);
    return 0;
}
```

---

## 简单计时器

```cpp
#include <boost/timer/progress_display.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    const int total = 100;

    boost::timer::progress_display progress(total);

    for (int i = 0; i < total; ++i) {
        // 模拟工作
        std::this_thread::sleep_for(std::chrono::milliseconds(50));
        ++progress;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Timer 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/timer/doc/index.html)

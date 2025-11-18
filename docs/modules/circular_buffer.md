# Boost.CircularBuffer - 循环缓冲区库

## 概述

Boost.CircularBuffer 提供固定大小的循环缓冲区，当缓冲区满时自动覆盖最旧的元素。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    // 创建容量为5的循环缓冲区
    boost::circular_buffer<int> cb(5);

    // 添加元素
    cb.push_back(1);
    cb.push_back(2);
    cb.push_back(3);

    std::cout << "缓冲区内容: ";
    for (int x : cb) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    std::cout << "大小: " << cb.size() << std::endl;
    std::cout << "容量: " << cb.capacity() << std::endl;

    return 0;
}
```

---

## 基本操作

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    boost::circular_buffer<int> cb(5);

    // push_back
    for (int i = 1; i <= 7; ++i) {
        cb.push_back(i);
        std::cout << "添加 " << i << ": ";
        for (int x : cb) {
            std::cout << x << " ";
        }
        std::cout << std::endl;
    }

    // 注意：只保留最后5个元素（3,4,5,6,7）

    return 0;
}
```

---

## 前后访问

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    boost::circular_buffer<int> cb(5);

    for (int i = 1; i <= 5; ++i) {
        cb.push_back(i);
    }

    // 访问元素
    std::cout << "第一个元素: " << cb.front() << std::endl;
    std::cout << "最后元素: " << cb.back() << std::endl;
    std::cout << "索引[2]: " << cb[2] << std::endl;

    // push_front
    cb.push_front(0);
    std::cout << "前端添加0后: ";
    for (int x : cb) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // pop_back
    cb.pop_back();
    std::cout << "后端删除后: ";
    for (int x : cb) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 迭代器

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <algorithm>

int main() {
    boost::circular_buffer<int> cb(10);

    for (int i = 1; i <= 10; ++i) {
        cb.push_back(i);
    }

    // 正向迭代
    std::cout << "正向: ";
    for (auto it = cb.begin(); it != cb.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 反向迭代
    std::cout << "反向: ";
    for (auto it = cb.rbegin(); it != cb.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 使用算法
    auto it = std::find(cb.begin(), cb.end(), 5);
    if (it != cb.end()) {
        std::cout << "找到5在位置: " << std::distance(cb.begin(), it) << std::endl;
    }

    return 0;
}
```

---

## 容量管理

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    boost::circular_buffer<int> cb(5);

    std::cout << "初始容量: " << cb.capacity() << std::endl;
    std::cout << "初始大小: " << cb.size() << std::endl;
    std::cout << "是否为空: " << std::boolalpha << cb.empty() << std::endl;
    std::cout << "是否已满: " << cb.full() << std::endl;

    for (int i = 1; i <= 5; ++i) {
        cb.push_back(i);
    }

    std::cout << "\n添加5个元素后:" << std::endl;
    std::cout << "大小: " << cb.size() << std::endl;
    std::cout << "是否已满: " << cb.full() << std::endl;

    // 调整容量
    cb.set_capacity(10);
    std::cout << "\n调整容量后: " << cb.capacity() << std::endl;

    return 0;
}
```

---

## 滑动窗口

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <numeric>

int main() {
    const int window_size = 5;
    boost::circular_buffer<double> window(window_size);

    std::vector<double> data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    std::cout << "滑动窗口平均值:\n";
    for (double value : data) {
        window.push_back(value);

        if (window.full()) {
            double sum = std::accumulate(window.begin(), window.end(), 0.0);
            double avg = sum / window.size();
            std::cout << "  数据点 " << value << ": 平均 = " << avg << std::endl;
        }
    }

    return 0;
}
```

---

## 日志缓冲

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <string>
#include <chrono>
#include <ctime>

struct LogEntry {
    std::string message;
    std::time_t timestamp;

    LogEntry(const std::string& msg)
        : message(msg), timestamp(std::time(nullptr)) {}
};

class Logger {
public:
    Logger(size_t capacity) : buffer_(capacity) {}

    void log(const std::string& message) {
        buffer_.push_back(LogEntry(message));
    }

    void print_recent() {
        std::cout << "最近的日志:\n";
        for (const auto& entry : buffer_) {
            char time_str[100];
            std::strftime(time_str, sizeof(time_str), "%Y-%m-%d %H:%M:%S",
                         std::localtime(&entry.timestamp));
            std::cout << "  [" << time_str << "] " << entry.message << std::endl;
        }
    }

private:
    boost::circular_buffer<LogEntry> buffer_;
};

int main() {
    Logger logger(5);  // 只保留最近5条日志

    logger.log("应用程序启动");
    logger.log("连接数据库");
    logger.log("加载配置");
    logger.log("启动服务器");
    logger.log("准备就绪");
    logger.log("接收请求");  // 覆盖"应用程序启动"
    logger.log("处理请求");  // 覆盖"连接数据库"

    logger.print_recent();

    return 0;
}
```

---

## 性能监控

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <random>
#include <numeric>

struct Metric {
    double cpu_usage;
    double memory_usage;
};

int main() {
    boost::circular_buffer<Metric> metrics(60);  // 保留60秒的数据

    // 模拟数据收集
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_real_distribution<> cpu_dist(10.0, 90.0);
    std::uniform_real_distribution<> mem_dist(30.0, 80.0);

    for (int i = 0; i < 100; ++i) {
        metrics.push_back({cpu_dist(gen), mem_dist(gen)});
    }

    // 计算统计
    double avg_cpu = std::accumulate(metrics.begin(), metrics.end(), 0.0,
        [](double sum, const Metric& m) { return sum + m.cpu_usage; }) / metrics.size();

    double avg_mem = std::accumulate(metrics.begin(), metrics.end(), 0.0,
        [](double sum, const Metric& m) { return sum + m.memory_usage; }) / metrics.size();

    std::cout << "最近60秒平均值:\n";
    std::cout << "  CPU: " << avg_cpu << "%\n";
    std::cout << "  内存: " << avg_mem << "%\n";

    return 0;
}
```

---

## 固定大小队列

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <string>

template<typename T>
class FixedQueue {
public:
    FixedQueue(size_t capacity) : buffer_(capacity) {}

    void push(const T& item) {
        buffer_.push_back(item);
    }

    T pop() {
        if (buffer_.empty()) {
            throw std::runtime_error("队列为空");
        }
        T item = buffer_.front();
        buffer_.pop_front();
        return item;
    }

    bool empty() const {
        return buffer_.empty();
    }

    size_t size() const {
        return buffer_.size();
    }

private:
    boost::circular_buffer<T> buffer_;
};

int main() {
    FixedQueue<std::string> queue(5);

    // 添加元素
    for (int i = 1; i <= 7; ++i) {
        queue.push("任务" + std::to_string(i));
        std::cout << "添加任务" << i << ", 队列大小: " << queue.size() << std::endl;
    }

    // 处理元素
    std::cout << "\n处理任务:\n";
    while (!queue.empty()) {
        std::cout << "  处理: " << queue.pop() << std::endl;
    }

    return 0;
}
```

---

## 清空和重置

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    boost::circular_buffer<int> cb(5);

    for (int i = 1; i <= 5; ++i) {
        cb.push_back(i);
    }

    std::cout << "初始: ";
    for (int x : cb) std::cout << x << " ";
    std::cout << std::endl;

    // 清空
    cb.clear();
    std::cout << "清空后大小: " << cb.size() << std::endl;
    std::cout << "清空后容量: " << cb.capacity() << std::endl;

    // 重新添加
    for (int i = 10; i <= 15; ++i) {
        cb.push_back(i);
    }

    std::cout << "重新添加: ";
    for (int x : cb) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.CircularBuffer 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/circular_buffer.html)

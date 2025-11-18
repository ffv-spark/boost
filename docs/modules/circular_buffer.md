# Boost.Circular_Buffer - 循环缓冲区

## 概述

Boost.Circular_Buffer 提供固定大小的循环缓冲区，自动覆盖最旧的元素。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>

int main() {
    // 创建容量为3的循环缓冲区
    boost::circular_buffer<int> cb(3);

    cb.push_back(1);
    cb.push_back(2);
    cb.push_back(3);

    std::cout << "Buffer: ";
    for (int x : cb) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 添加第4个元素，覆盖第1个
    cb.push_back(4);

    std::cout << "After push: ";
    for (int x : cb) {
        std::cout << x << " ";  // 输出: 2 3 4
    }
    std::cout << std::endl;

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

    // 添加元素
    cb.push_back(1);
    cb.push_back(2);
    cb.push_front(0);  // [0, 1, 2]

    // 访问元素
    std::cout << "Front: " << cb.front() << std::endl;  // 0
    std::cout << "Back: " << cb.back() << std::endl;   // 2
    std::cout << "At[1]: " << cb[1] << std::endl;      // 1

    // 大小信息
    std::cout << "Size: " << cb.size() << std::endl;
    std::cout << "Capacity: " << cb.capacity() << std::endl;
    std::cout << "Full: " << cb.full() << std::endl;
    std::cout << "Empty: " << cb.empty() << std::endl;

    // 删除元素
    cb.pop_front();  // [1, 2]
    cb.pop_back();   // [1]

    return 0;
}
```

---

## 实用示例

### 日志缓冲

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <string>
#include <ctime>

class LogBuffer {
public:
    LogBuffer(size_t capacity) : buffer_(capacity) {}

    void add_log(const std::string& message) {
        std::time_t now = std::time(nullptr);
        char timestamp[20];
        std::strftime(timestamp, sizeof(timestamp), "%Y-%m-%d %H:%M:%S",
                      std::localtime(&now));

        std::string log_entry = std::string(timestamp) + " - " + message;
        buffer_.push_back(log_entry);
    }

    void print_recent_logs(size_t count = 0) {
        if (count == 0 || count > buffer_.size()) {
            count = buffer_.size();
        }

        std::cout << "最近 " << count << " 条日志:\n";
        auto it = buffer_.end() - count;
        for (; it != buffer_.end(); ++it) {
            std::cout << *it << std::endl;
        }
    }

private:
    boost::circular_buffer<std::string> buffer_;
};

int main() {
    LogBuffer logger(5);  // 只保留最近5条日志

    for (int i = 1; i <= 8; ++i) {
        logger.add_log("Event " + std::to_string(i));
    }

    logger.print_recent_logs();

    return 0;
}
```

### 移动平均

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <numeric>

class MovingAverage {
public:
    MovingAverage(size_t window_size) : buffer_(window_size) {}

    void add_value(double value) {
        buffer_.push_back(value);
    }

    double get_average() const {
        if (buffer_.empty()) {
            return 0.0;
        }
        double sum = std::accumulate(buffer_.begin(), buffer_.end(), 0.0);
        return sum / buffer_.size();
    }

private:
    boost::circular_buffer<double> buffer_;
};

int main() {
    MovingAverage ma(5);  // 5个数据的移动平均

    std::vector<double> data = {10, 20, 30, 40, 50, 60, 70, 80};

    for (double value : data) {
        ma.add_value(value);
        std::cout << "Value: " << value
                  << ", Moving Avg: " << ma.get_average()
                  << std::endl;
    }

    return 0;
}
```

### 最近访问记录

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <string>

class RecentHistory {
public:
    RecentHistory(size_t size) : history_(size) {}

    void add(const std::string& item) {
        // 避免重复
        auto it = std::find(history_.begin(), history_.end(), item);
        if (it != history_.end()) {
            history_.erase(it);
        }
        history_.push_back(item);
    }

    void print() const {
        std::cout << "最近访问:\n";
        for (auto it = history_.rbegin(); it != history_.rend(); ++it) {
            std::cout << "  - " << *it << std::endl;
        }
    }

private:
    boost::circular_buffer<std::string> history_;
};

int main() {
    RecentHistory history(5);

    history.add("index.html");
    history.add("about.html");
    history.add("contact.html");
    history.add("index.html");  // 重复访问
    history.add("blog.html");
    history.add("products.html");

    history.print();

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
    boost::circular_buffer<int> cb(5);

    for (int i = 1; i <= 7; ++i) {
        cb.push_back(i);
    }

    // 正向遍历
    std::cout << "Forward: ";
    for (auto it = cb.begin(); it != cb.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 反向遍历
    std::cout << "Reverse: ";
    for (auto it = cb.rbegin(); it != cb.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 使用算法
    auto max_it = std::max_element(cb.begin(), cb.end());
    std::cout << "Max: " << *max_it << std::endl;

    return 0;
}
```

---

## 性能特性

```cpp
#include <boost/circular_buffer.hpp>
#include <iostream>
#include <chrono>

int main() {
    const size_t size = 1000000;
    boost::circular_buffer<int> cb(size);

    auto start = std::chrono::high_resolution_clock::now();

    for (int i = 0; i < size * 2; ++i) {
        cb.push_back(i);
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "插入 " << (size * 2) << " 个元素耗时: "
              << duration.count() << " ms" << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **固定容量**: 适合需要固定大小缓冲区的场景
2. **性能**: O(1) 插入和删除
3. **线程安全**: 需要外部同步
4. **内存**: 预分配固定内存
5. **用途**: 日志、历史记录、滑动窗口

---

## 参考资源

- [Boost.Circular_Buffer 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/circular_buffer.html)

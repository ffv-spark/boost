# Boost.Thread - 多线程库

## 概述

Boost.Thread 提供了跨平台的线程管理、同步原语和线程安全工具。

**类型**: 需要编译链接的库

**链接库**: `-lboost_thread -lboost_system -lpthread`

**注意**: C++11 已将大部分功能纳入标准库 (`std::thread`)

---

## 快速开始

```cpp
#include <boost/thread.hpp>
#include <iostream>

void hello() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    boost::thread t(hello);
    t.join();
    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_thread -lboost_system -lpthread -o example
```

---

## 线程创建和管理

### 基本线程操作

```cpp
#include <boost/thread.hpp>
#include <iostream>

void thread_function(int id) {
    std::cout << "线程 " << id << " 正在运行" << std::endl;
    boost::this_thread::sleep_for(boost::chrono::seconds(1));
    std::cout << "线程 " << id << " 完成" << std::endl;
}

int main() {
    // 1. 创建线程
    boost::thread t1(thread_function, 1);
    boost::thread t2(thread_function, 2);

    // 2. 等待线程完成
    t1.join();
    t2.join();

    // 3. 分离线程（不等待）
    boost::thread t3(thread_function, 3);
    t3.detach();

    boost::this_thread::sleep_for(boost::chrono::seconds(2));

    return 0;
}
```

### 使用 Lambda 表达式

```cpp
#include <boost/thread.hpp>
#include <iostream>

int main() {
    int value = 42;

    // Lambda 线程
    boost::thread t([&value]() {
        std::cout << "Value: " << value << std::endl;
        value += 10;
    });

    t.join();

    std::cout << "Updated value: " << value << std::endl;

    return 0;
}
```

### 线程组

```cpp
#include <boost/thread.hpp>
#include <iostream>

void worker(int id) {
    std::cout << "Worker " << id << " start" << std::endl;
    boost::this_thread::sleep_for(boost::chrono::seconds(1));
    std::cout << "Worker " << id << " done" << std::endl;
}

int main() {
    boost::thread_group threads;

    // 创建多个线程
    for (int i = 0; i < 5; ++i) {
        threads.create_thread(boost::bind(worker, i));
    }

    // 等待所有线程完成
    threads.join_all();

    return 0;
}
```

---

## 互斥锁（Mutex）

### 基本互斥锁

```cpp
#include <boost/thread.hpp>
#include <iostream>

boost::mutex mtx;
int counter = 0;

void increment(int id, int count) {
    for (int i = 0; i < count; ++i) {
        boost::lock_guard<boost::mutex> lock(mtx);
        ++counter;
        std::cout << "Thread " << id << ": counter = " << counter << std::endl;
    }
}

int main() {
    boost::thread t1(increment, 1, 5);
    boost::thread t2(increment, 2, 5);

    t1.join();
    t2.join();

    std::cout << "Final counter: " << counter << std::endl;

    return 0;
}
```

### 递归互斥锁

```cpp
#include <boost/thread.hpp>
#include <iostream>

class Counter {
public:
    Counter() : count_(0) {}

    void increment() {
        boost::lock_guard<boost::recursive_mutex> lock(mutex_);
        ++count_;
        if (count_ < 5) {
            increment(); // 递归调用
        }
    }

    int get() {
        boost::lock_guard<boost::recursive_mutex> lock(mutex_);
        return count_;
    }

private:
    boost::recursive_mutex mutex_;
    int count_;
};

int main() {
    Counter counter;
    counter.increment();
    std::cout << "Count: " << counter.get() << std::endl;

    return 0;
}
```

### 读写锁

```cpp
#include <boost/thread.hpp>
#include <boost/thread/shared_mutex.hpp>
#include <iostream>
#include <vector>

class ThreadSafeCounter {
public:
    ThreadSafeCounter() : value_(0) {}

    // 读操作（共享锁）
    int get() const {
        boost::shared_lock<boost::shared_mutex> lock(mutex_);
        return value_;
    }

    // 写操作（独占锁）
    void increment() {
        boost::unique_lock<boost::shared_mutex> lock(mutex_);
        ++value_;
    }

private:
    mutable boost::shared_mutex mutex_;
    int value_;
};

int main() {
    ThreadSafeCounter counter;

    // 多个读线程
    boost::thread_group readers;
    for (int i = 0; i < 5; ++i) {
        readers.create_thread([&counter, i]() {
            for (int j = 0; j < 10; ++j) {
                std::cout << "Reader " << i << ": " << counter.get() << std::endl;
                boost::this_thread::sleep_for(boost::chrono::milliseconds(10));
            }
        });
    }

    // 一个写线程
    boost::thread writer([&counter]() {
        for (int i = 0; i < 50; ++i) {
            counter.increment();
            boost::this_thread::sleep_for(boost::chrono::milliseconds(20));
        }
    });

    readers.join_all();
    writer.join();

    std::cout << "Final value: " << counter.get() << std::endl;

    return 0;
}
```

---

## 条件变量

### 生产者-消费者模式

```cpp
#include <boost/thread.hpp>
#include <boost/thread/condition_variable.hpp>
#include <iostream>
#include <queue>

class MessageQueue {
public:
    void push(int value) {
        boost::lock_guard<boost::mutex> lock(mutex_);
        queue_.push(value);
        cond_.notify_one();
    }

    int pop() {
        boost::unique_lock<boost::mutex> lock(mutex_);

        // 等待直到队列非空
        cond_.wait(lock, [this]() { return !queue_.empty(); });

        int value = queue_.front();
        queue_.pop();
        return value;
    }

private:
    boost::mutex mutex_;
    boost::condition_variable cond_;
    std::queue<int> queue_;
};

void producer(MessageQueue& mq, int id) {
    for (int i = 0; i < 5; ++i) {
        int value = id * 100 + i;
        mq.push(value);
        std::cout << "Producer " << id << " pushed: " << value << std::endl;
        boost::this_thread::sleep_for(boost::chrono::milliseconds(100));
    }
}

void consumer(MessageQueue& mq, int id) {
    for (int i = 0; i < 5; ++i) {
        int value = mq.pop();
        std::cout << "Consumer " << id << " popped: " << value << std::endl;
    }
}

int main() {
    MessageQueue mq;

    boost::thread p1(producer, boost::ref(mq), 1);
    boost::thread p2(producer, boost::ref(mq), 2);
    boost::thread c1(consumer, boost::ref(mq), 1);
    boost::thread c2(consumer, boost::ref(mq), 2);

    p1.join();
    p2.join();
    c1.join();
    c2.join();

    return 0;
}
```

### 超时等待

```cpp
#include <boost/thread.hpp>
#include <iostream>

boost::mutex mtx;
boost::condition_variable cv;
bool ready = false;

void wait_for_signal() {
    boost::unique_lock<boost::mutex> lock(mtx);

    // 等待最多 3 秒
    bool result = cv.wait_for(
        lock,
        boost::chrono::seconds(3),
        []() { return ready; }
    );

    if (result) {
        std::cout << "收到信号！" << std::endl;
    } else {
        std::cout << "等待超时" << std::endl;
    }
}

int main() {
    boost::thread t(wait_for_signal);

    // 2 秒后发送信号
    boost::this_thread::sleep_for(boost::chrono::seconds(2));

    {
        boost::lock_guard<boost::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();

    t.join();

    return 0;
}
```

---

## 线程局部存储

```cpp
#include <boost/thread.hpp>
#include <iostream>

// 每个线程都有自己的副本
boost::thread_specific_ptr<int> thread_local_value;

void thread_function(int id) {
    // 为当前线程设置值
    thread_local_value.reset(new int(id * 10));

    std::cout << "Thread " << id
              << " local value: " << *thread_local_value << std::endl;

    boost::this_thread::sleep_for(boost::chrono::seconds(1));

    std::cout << "Thread " << id
              << " still has: " << *thread_local_value << std::endl;
}

int main() {
    boost::thread t1(thread_function, 1);
    boost::thread t2(thread_function, 2);
    boost::thread t3(thread_function, 3);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

---

## Future 和 Promise

### 基本 Future

```cpp
#include <boost/thread.hpp>
#include <boost/thread/future.hpp>
#include <iostream>

int calculate(int x) {
    boost::this_thread::sleep_for(boost::chrono::seconds(2));
    return x * x;
}

int main() {
    // 使用 async 启动异步任务
    boost::future<int> fut = boost::async(boost::launch::async, calculate, 10);

    std::cout << "计算中..." << std::endl;

    // 获取结果（阻塞）
    int result = fut.get();

    std::cout << "结果: " << result << std::endl;

    return 0;
}
```

### Promise 和 Future

```cpp
#include <boost/thread.hpp>
#include <boost/thread/future.hpp>
#include <iostream>

void async_task(boost::promise<int>& prom, int value) {
    boost::this_thread::sleep_for(boost::chrono::seconds(1));

    try {
        if (value < 0) {
            throw std::runtime_error("Negative value!");
        }
        prom.set_value(value * 2);
    } catch (...) {
        prom.set_exception(boost::current_exception());
    }
}

int main() {
    boost::promise<int> prom;
    boost::future<int> fut = prom.get_future();

    boost::thread t(async_task, boost::ref(prom), 21);

    std::cout << "等待结果..." << std::endl;

    try {
        int result = fut.get();
        std::cout << "结果: " << result << std::endl;
    } catch (const std::exception& e) {
        std::cout << "异常: " << e.what() << std::endl;
    }

    t.join();

    return 0;
}
```

### Future Continuations

```cpp
#include <boost/thread.hpp>
#include <boost/thread/future.hpp>
#include <iostream>

int main() {
    // 创建异步任务
    auto fut1 = boost::async([]() {
        std::cout << "Task 1" << std::endl;
        return 42;
    });

    // 添加延续任务
    auto fut2 = fut1.then([](boost::future<int> f) {
        int value = f.get();
        std::cout << "Task 2, got: " << value << std::endl;
        return value * 2;
    });

    auto fut3 = fut2.then([](boost::future<int> f) {
        int value = f.get();
        std::cout << "Task 3, got: " << value << std::endl;
        return value + 10;
    });

    std::cout << "最终结果: " << fut3.get() << std::endl;

    return 0;
}
```

---

## 线程池

```cpp
#include <boost/asio/thread_pool.hpp>
#include <boost/asio/post.hpp>
#include <iostream>
#include <vector>

void task(int id) {
    std::cout << "Task " << id << " on thread "
              << boost::this_thread::get_id() << std::endl;
    boost::this_thread::sleep_for(boost::chrono::milliseconds(100));
}

int main() {
    // 创建 4 线程的线程池
    boost::asio::thread_pool pool(4);

    // 提交任务
    for (int i = 0; i < 20; ++i) {
        boost::asio::post(pool, boost::bind(task, i));
    }

    // 等待所有任务完成
    pool.join();

    return 0;
}
```

---

## 实用示例

### 并行计算

```cpp
#include <boost/thread.hpp>
#include <iostream>
#include <vector>
#include <numeric>

// 并行求和
long long parallel_sum(const std::vector<int>& data, int num_threads) {
    std::vector<long long> partial_sums(num_threads, 0);
    boost::thread_group threads;

    size_t chunk_size = data.size() / num_threads;

    for (int i = 0; i < num_threads; ++i) {
        size_t start = i * chunk_size;
        size_t end = (i == num_threads - 1) ? data.size() : (i + 1) * chunk_size;

        threads.create_thread([&data, &partial_sums, i, start, end]() {
            partial_sums[i] = std::accumulate(
                data.begin() + start,
                data.begin() + end,
                0LL
            );
        });
    }

    threads.join_all();

    return std::accumulate(partial_sums.begin(), partial_sums.end(), 0LL);
}

int main() {
    std::vector<int> data(10000000, 1);

    auto start = boost::chrono::high_resolution_clock::now();
    long long sum = parallel_sum(data, 4);
    auto end = boost::chrono::high_resolution_clock::now();

    std::cout << "Sum: " << sum << std::endl;
    std::cout << "Time: "
              << boost::chrono::duration_cast<boost::chrono::milliseconds>(end - start).count()
              << " ms" << std::endl;

    return 0;
}
```

### 定时任务

```cpp
#include <boost/thread.hpp>
#include <iostream>

class TimerTask {
public:
    TimerTask(int interval_ms, std::function<void()> func)
        : interval_(interval_ms)
        , func_(func)
        , running_(true)
        , thread_(&TimerTask::run, this) {
    }

    ~TimerTask() {
        stop();
    }

    void stop() {
        running_ = false;
        if (thread_.joinable()) {
            thread_.join();
        }
    }

private:
    void run() {
        while (running_) {
            boost::this_thread::sleep_for(
                boost::chrono::milliseconds(interval_)
            );
            if (running_) {
                func_();
            }
        }
    }

    int interval_;
    std::function<void()> func_;
    bool running_;
    boost::thread thread_;
};

int main() {
    int count = 0;

    TimerTask timer(1000, [&count]() {
        std::cout << "Tick " << ++count << std::endl;
    });

    boost::this_thread::sleep_for(boost::chrono::seconds(5));

    timer.stop();

    return 0;
}
```

---

## 最佳实践

1. **避免数据竞争**: 使用互斥锁保护共享数据
2. **避免死锁**: 始终以相同顺序获取多个锁
3. **使用 RAII**: 使用 `lock_guard` 而不是手动 lock/unlock
4. **选择合适的锁**: 读多写少时使用 `shared_mutex`
5. **异常安全**: 使用 RAII 确保异常时正确释放锁

---

## 常见陷阱

```cpp
// ❌ 错误：忘记 join 或 detach
void bad_example1() {
    boost::thread t([]() { /* ... */ });
    // 线程对象销毁时，如果没有 join 或 detach，程序会终止
}

// ❌ 错误：悬垂引用
void bad_example2() {
    int value = 42;
    boost::thread t([&value]() {
        boost::this_thread::sleep_for(boost::chrono::seconds(1));
        std::cout << value << std::endl; // value 可能已被销毁
    });
    t.detach();
    // value 在函数结束时销毁，但线程可能还在运行
}

// ✅ 正确：传值或使用智能指针
void good_example() {
    auto value = std::make_shared<int>(42);
    boost::thread t([value]() {
        boost::this_thread::sleep_for(boost::chrono::seconds(1));
        std::cout << *value << std::endl;
    });
    t.detach();
}
```

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_thread -lboost_system -lpthread -o example

# CMake
find_package(Boost REQUIRED COMPONENTS thread system)
target_link_libraries(myapp Boost::thread Boost::system)
```

---

## 参考资源

- [Boost.Thread 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/thread.html)
- [C++ 并发编程最佳实践](https://www.boost.org/doc/libs/1_90_0/doc/html/thread/best_practices.html)

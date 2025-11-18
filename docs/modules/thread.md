# Boost.Thread - 线程库

## 概述

Boost.Thread 提供可移植的多线程编程支持，是 C++11 std::thread 的前身。

**类型**: 需要编译的库

**注意**: C++11 引入了 `<thread>`，优先使用标准库版本

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

**编译**: `g++ -std=c++14 example.cpp -lboost_thread -lboost_system -lpthread`

---

## 创建线程

```cpp
#include <boost/thread.hpp>
#include <iostream>

void print_numbers(int n) {
    for (int i = 0; i < n; ++i) {
        std::cout << "线程: " << i << std::endl;
    }
}

int main() {
    // 函数指针
    boost::thread t1(print_numbers, 5);

    // Lambda
    boost::thread t2([]() {
        std::cout << "Lambda 线程" << std::endl;
    });

    // 函数对象
    struct Functor {
        void operator()() {
            std::cout << "函数对象线程" << std::endl;
        }
    };
    boost::thread t3(Functor());

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

---

## 互斥锁

```cpp
#include <boost/thread.hpp>
#include <iostream>

boost::mutex mtx;
int shared_counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        boost::lock_guard<boost::mutex> lock(mtx);
        ++shared_counter;
    }
}

int main() {
    boost::thread t1(increment);
    boost::thread t2(increment);
    boost::thread t3(increment);

    t1.join();
    t2.join();
    t3.join();

    std::cout << "计数器: " << shared_counter << std::endl;

    return 0;
}
```

---

## 条件变量

```cpp
#include <boost/thread.hpp>
#include <iostream>
#include <queue>

boost::mutex mtx;
boost::condition_variable cv;
std::queue<int> data_queue;
bool done = false;

void producer() {
    for (int i = 0; i < 10; ++i) {
        boost::lock_guard<boost::mutex> lock(mtx);
        data_queue.push(i);
        std::cout << "生产: " << i << std::endl;
        cv.notify_one();
        boost::this_thread::sleep_for(boost::chrono::milliseconds(100));
    }

    {
        boost::lock_guard<boost::mutex> lock(mtx);
        done = true;
        cv.notify_all();
    }
}

void consumer() {
    while (true) {
        boost::unique_lock<boost::mutex> lock(mtx);
        cv.wait(lock, [] { return !data_queue.empty() || done; });

        while (!data_queue.empty()) {
            int value = data_queue.front();
            data_queue.pop();
            lock.unlock();

            std::cout << "消费: " << value << std::endl;
            boost::this_thread::sleep_for(boost::chrono::milliseconds(150));

            lock.lock();
        }

        if (done && data_queue.empty()) {
            break;
        }
    }
}

int main() {
    boost::thread prod(producer);
    boost::thread cons1(consumer);
    boost::thread cons2(consumer);

    prod.join();
    cons1.join();
    cons2.join();

    return 0;
}
```

---

## 读写锁

```cpp
#include <boost/thread.hpp>
#include <iostream>

boost::shared_mutex rw_mutex;
int shared_data = 0;

void reader(int id) {
    for (int i = 0; i < 5; ++i) {
        boost::shared_lock<boost::shared_mutex> lock(rw_mutex);
        std::cout << "读者 " << id << " 读取: " << shared_data << std::endl;
        boost::this_thread::sleep_for(boost::chrono::milliseconds(100));
    }
}

void writer(int id) {
    for (int i = 0; i < 5; ++i) {
        boost::unique_lock<boost::shared_mutex> lock(rw_mutex);
        ++shared_data;
        std::cout << "写者 " << id << " 写入: " << shared_data << std::endl;
        boost::this_thread::sleep_for(boost::chrono::milliseconds(200));
    }
}

int main() {
    boost::thread w1(writer, 1);
    boost::thread r1(reader, 1);
    boost::thread r2(reader, 2);
    boost::thread r3(reader, 3);

    w1.join();
    r1.join();
    r2.join();
    r3.join();

    return 0;
}
```

---

## 线程组

```cpp
#include <boost/thread.hpp>
#include <iostream>

void task(int id) {
    std::cout << "任务 " << id << " 开始" << std::endl;
    boost::this_thread::sleep_for(boost::chrono::seconds(1));
    std::cout << "任务 " << id << " 完成" << std::endl;
}

int main() {
    boost::thread_group threads;

    // 创建多个线程
    for (int i = 0; i < 5; ++i) {
        threads.create_thread([i]() { task(i); });
    }

    // 等待所有线程完成
    threads.join_all();

    std::cout << "所有任务完成" << std::endl;

    return 0;
}
```

---

## Future 和 Promise

```cpp
#include <boost/thread.hpp>
#include <iostream>

int calculate(int x) {
    boost::this_thread::sleep_for(boost::chrono::seconds(2));
    return x * x;
}

int main() {
    // 使用 async
    boost::future<int> future = boost::async(boost::launch::async, calculate, 10);

    std::cout << "等待结果..." << std::endl;
    int result = future.get();
    std::cout << "结果: " << result << std::endl;

    // 使用 promise
    boost::promise<int> promise;
    boost::future<int> future2 = promise.get_future();

    boost::thread t([&promise]() {
        boost::this_thread::sleep_for(boost::chrono::seconds(1));
        promise.set_value(42);
    });

    std::cout << "等待 promise..." << std::endl;
    std::cout << "Promise 结果: " << future2.get() << std::endl;

    t.join();

    return 0;
}
```

---

## 线程局部存储

```cpp
#include <boost/thread.hpp>
#include <iostream>

boost::thread_specific_ptr<int> thread_local_value;

void task(int id) {
    // 每个线程有自己的值
    thread_local_value.reset(new int(id * 10));

    std::cout << "线程 " << id << " 的值: " 
              << *thread_local_value << std::endl;

    boost::this_thread::sleep_for(boost::chrono::seconds(1));

    std::cout << "线程 " << id << " 仍然是: " 
              << *thread_local_value << std::endl;
}

int main() {
    boost::thread t1(task, 1);
    boost::thread t2(task, 2);
    boost::thread t3(task, 3);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

---

## 线程池

```cpp
#include <boost/thread.hpp>
#include <boost/asio.hpp>
#include <iostream>
#include <vector>

int main() {
    boost::asio::io_context io;
    boost::asio::io_context::work work(io);

    // 创建线程池
    std::vector<boost::thread> threads;
    for (int i = 0; i < 4; ++i) {
        threads.emplace_back([&io]() { io.run(); });
    }

    // 提交任务
    for (int i = 0; i < 10; ++i) {
        io.post([i]() {
            std::cout << "任务 " << i << " 在线程 " 
                      << boost::this_thread::get_id() << std::endl;
            boost::this_thread::sleep_for(boost::chrono::milliseconds(500));
        });
    }

    // 等待所有任务完成
    io.stop();
    for (auto& t : threads) {
        t.join();
    }

    return 0;
}
```

---

## 栅栏

```cpp
#include <boost/thread.hpp>
#include <iostream>

void phase_work(int id, boost::barrier& bar, int phase) {
    std::cout << "线程 " << id << " 开始阶段 " << phase << std::endl;
    boost::this_thread::sleep_for(boost::chrono::milliseconds(100 * id));
    std::cout << "线程 " << id << " 完成阶段 " << phase << std::endl;

    bar.wait();  // 等待所有线程
}

int main() {
    const int num_threads = 3;
    boost::barrier bar(num_threads);

    boost::thread_group threads;

    for (int i = 0; i < num_threads; ++i) {
        threads.create_thread([i, &bar]() {
            for (int phase = 1; phase <= 3; ++phase) {
                phase_work(i, bar, phase);
            }
        });
    }

    threads.join_all();

    return 0;
}
```

---

## 参考资源

- [Boost.Thread 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/thread.html)
- [C++11 `<thread>` 参考](https://en.cppreference.com/w/cpp/thread)

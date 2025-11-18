# Boost.Fiber - 纤程库

## 概述

Boost.Fiber 提供用户态线程（纤程），实现轻量级的协作式多任务，比操作系统线程更高效。

**类型**: 需要编译链接的库

**链接库**: `-lboost_fiber -lboost_context`

---

## 快速开始

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>

namespace fibers = boost::fibers;

void fiber_function(int id) {
    for (int i = 0; i < 3; ++i) {
        std::cout << "纤程 " << id << " - 迭代 " << i << std::endl;
        fibers::this_fiber::yield();  // 让出执行权
    }
}

int main() {
    fibers::fiber f1(fiber_function, 1);
    fibers::fiber f2(fiber_function, 2);

    f1.join();
    f2.join();

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_fiber -lboost_context -lpthread -o example
```

---

## 纤程同步 - 互斥锁

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>

namespace fibers = boost::fibers;

fibers::mutex mtx;
int shared_counter = 0;

void increment(int id, int count) {
    for (int i = 0; i < count; ++i) {
        std::lock_guard<fibers::mutex> lock(mtx);
        ++shared_counter;
        std::cout << "纤程 " << id << " 增加计数到 "
                  << shared_counter << std::endl;
        fibers::this_fiber::yield();
    }
}

int main() {
    fibers::fiber f1(increment, 1, 5);
    fibers::fiber f2(increment, 2, 5);

    f1.join();
    f2.join();

    std::cout << "最终计数: " << shared_counter << std::endl;

    return 0;
}
```

---

## 条件变量

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>
#include <queue>

namespace fibers = boost::fibers;

fibers::mutex mtx;
fibers::condition_variable cond;
std::queue<int> data_queue;
bool done = false;

void producer() {
    for (int i = 1; i <= 10; ++i) {
        std::unique_lock<fibers::mutex> lock(mtx);
        data_queue.push(i);
        std::cout << "生产: " << i << std::endl;
        lock.unlock();
        cond.notify_one();
        fibers::this_fiber::sleep_for(std::chrono::milliseconds(100));
    }

    std::unique_lock<fibers::mutex> lock(mtx);
    done = true;
    cond.notify_all();
}

void consumer(int id) {
    while (true) {
        std::unique_lock<fibers::mutex> lock(mtx);
        cond.wait(lock, []{ return !data_queue.empty() || done; });

        if (data_queue.empty() && done) {
            break;
        }

        if (!data_queue.empty()) {
            int value = data_queue.front();
            data_queue.pop();
            lock.unlock();
            std::cout << "消费者 " << id << " 消费: " << value << std::endl;
        }
    }
}

int main() {
    fibers::fiber prod(producer);
    fibers::fiber cons1(consumer, 1);
    fibers::fiber cons2(consumer, 2);

    prod.join();
    cons1.join();
    cons2.join();

    return 0;
}
```

---

## 通道（Channel）

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>

namespace fibers = boost::fibers;

void sender(fibers::buffered_channel<int>& chan) {
    for (int i = 1; i <= 5; ++i) {
        chan.push(i);
        std::cout << "发送: " << i << std::endl;
        fibers::this_fiber::sleep_for(std::chrono::milliseconds(100));
    }
    chan.close();
}

void receiver(fibers::buffered_channel<int>& chan) {
    int value;
    while (fibers::channel_op_status::success == chan.pop(value)) {
        std::cout << "接收: " << value << std::endl;
    }
    std::cout << "通道已关闭" << std::endl;
}

int main() {
    fibers::buffered_channel<int> chan(10);

    fibers::fiber send_fiber(sender, std::ref(chan));
    fibers::fiber recv_fiber(receiver, std::ref(chan));

    send_fiber.join();
    recv_fiber.join();

    return 0;
}
```

---

## 纤程池

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>
#include <vector>
#include <functional>

namespace fibers = boost::fibers;

class FiberPool {
public:
    FiberPool(size_t num_fibers) {
        for (size_t i = 0; i < num_fibers; ++i) {
            workers_.emplace_back([this]{ worker_function(); });
        }
    }

    ~FiberPool() {
        {
            std::unique_lock<fibers::mutex> lock(mtx_);
            stop_ = true;
        }
        cond_.notify_all();

        for (auto& fiber : workers_) {
            fiber.join();
        }
    }

    template<typename F>
    void submit(F&& task) {
        {
            std::unique_lock<fibers::mutex> lock(mtx_);
            tasks_.push(std::forward<F>(task));
        }
        cond_.notify_one();
    }

private:
    void worker_function() {
        while (true) {
            std::function<void()> task;

            {
                std::unique_lock<fibers::mutex> lock(mtx_);
                cond_.wait(lock, [this]{ return stop_ || !tasks_.empty(); });

                if (stop_ && tasks_.empty()) {
                    return;
                }

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();
        }
    }

    std::vector<fibers::fiber> workers_;
    std::queue<std::function<void()>> tasks_;
    fibers::mutex mtx_;
    fibers::condition_variable cond_;
    bool stop_ = false;
};

int main() {
    FiberPool pool(4);

    for (int i = 0; i < 10; ++i) {
        pool.submit([i]{
            std::cout << "任务 " << i << " 开始" << std::endl;
            fibers::this_fiber::sleep_for(std::chrono::milliseconds(100));
            std::cout << "任务 " << i << " 完成" << std::endl;
        });
    }

    // 等待所有任务完成
    fibers::this_fiber::sleep_for(std::chrono::seconds(2));

    return 0;
}
```

---

## Future 和 Promise

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>

namespace fibers = boost::fibers;

int calculate(int x) {
    fibers::this_fiber::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main() {
    // 使用 packaged_task
    fibers::packaged_task<int(int)> task(calculate);
    fibers::future<int> result = task.get_future();

    fibers::fiber f(std::move(task), 5);

    std::cout << "等待结果..." << std::endl;
    std::cout << "结果: " << result.get() << std::endl;

    f.join();

    // 使用 async
    auto fut = fibers::async(fibers::launch::post, calculate, 10);
    std::cout << "异步结果: " << fut.get() << std::endl;

    return 0;
}
```

---

## 调度策略

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>

namespace fibers = boost::fibers;

void fiber_task(int id) {
    for (int i = 0; i < 3; ++i) {
        std::cout << "纤程 " << id << " - 迭代 " << i << std::endl;
        fibers::this_fiber::yield();
    }
}

int main() {
    // 使用轮询调度器
    fibers::use_scheduling_algorithm<fibers::algo::round_robin>();

    std::vector<fibers::fiber> fibers_vec;

    for (int i = 0; i < 5; ++i) {
        fibers_vec.emplace_back(fiber_task, i);
    }

    for (auto& f : fibers_vec) {
        f.join();
    }

    return 0;
}
```

---

## 工作窃取调度器

```cpp
#include <boost/fiber/all.hpp>
#include <iostream>
#include <thread>
#include <vector>

namespace fibers = boost::fibers;

void compute_task(int id) {
    std::cout << "任务 " << id << " 在线程 "
              << std::this_thread::get_id() << std::endl;
    fibers::this_fiber::sleep_for(std::chrono::milliseconds(100));
}

void thread_function(int num_fibers) {
    // 每个线程使用工作窃取调度器
    fibers::use_scheduling_algorithm<fibers::algo::work_stealing>(4);

    for (int i = 0; i < num_fibers; ++i) {
        fibers::fiber f(compute_task, i);
        f.detach();
    }

    // 让纤程运行
    fibers::this_fiber::sleep_for(std::chrono::seconds(1));
}

int main() {
    std::vector<std::thread> threads;

    // 创建多个线程，每个线程运行多个纤程
    for (int i = 0; i < 3; ++i) {
        threads.emplace_back(thread_function, 5);
    }

    for (auto& t : threads) {
        t.join();
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Fiber 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/fiber/doc/html/index.html)
- [纤程同步](https://www.boost.org/doc/libs/1_90_0/libs/fiber/doc/html/fiber/synchronization.html)
- [调度算法](https://www.boost.org/doc/libs/1_90_0/libs/fiber/doc/html/fiber/scheduling.html)

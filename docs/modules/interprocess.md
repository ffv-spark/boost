# Boost.Interprocess - 进程间通信库

## 概述

Boost.Interprocess 提供进程间通信（IPC）机制，包括共享内存、消息队列、信号量等。

**类型**: 仅头文件库

---

## 快速开始 - 共享内存

```cpp
#include <boost/interprocess/shared_memory_object.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>
#include <cstring>

namespace bip = boost::interprocess;

int main() {
    try {
        // 创建共享内存
        bip::shared_memory_object shm(
            bip::create_only,
            "MySharedMemory",
            bip::read_write
        );

        // 设置大小
        shm.truncate(1024);

        // 映射到进程地址空间
        bip::mapped_region region(shm, bip::read_write);

        // 写入数据
        std::strcpy(static_cast<char*>(region.get_address()), "Hello from shared memory!");

        std::cout << "写入共享内存成功" << std::endl;
        std::cout << "按回车键继续..." << std::endl;
        std::cin.get();

        // 清理
        bip::shared_memory_object::remove("MySharedMemory");
    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## 读取共享内存

```cpp
#include <boost/interprocess/shared_memory_object.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>

namespace bip = boost::interprocess;

int main() {
    try {
        // 打开已存在的共享内存
        bip::shared_memory_object shm(
            bip::open_only,
            "MySharedMemory",
            bip::read_only
        );

        // 映射到进程地址空间
        bip::mapped_region region(shm, bip::read_only);

        // 读取数据
        const char* data = static_cast<const char*>(region.get_address());
        std::cout << "读取共享内存: " << data << std::endl;

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## 共享内存中的对象

```cpp
#include <boost/interprocess/shared_memory_object.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>

namespace bip = boost::interprocess;

struct SharedData {
    int counter;
    double value;
    char message[256];
};

// 写入进程
void writer() {
    // 移除之前可能存在的共享内存
    bip::shared_memory_object::remove("SharedObject");

    // 创建共享内存
    bip::shared_memory_object shm(
        bip::create_only,
        "SharedObject",
        bip::read_write
    );

    shm.truncate(sizeof(SharedData));

    bip::mapped_region region(shm, bip::read_write);

    // 在共享内存中构造对象
    SharedData* data = new (region.get_address()) SharedData;
    data->counter = 42;
    data->value = 3.14159;
    std::strcpy(data->message, "Shared object data");

    std::cout << "写入对象完成" << std::endl;
    std::cout << "按回车键继续..." << std::endl;
    std::cin.get();
}

// 读取进程
void reader() {
    bip::shared_memory_object shm(
        bip::open_only,
        "SharedObject",
        bip::read_only
    );

    bip::mapped_region region(shm, bip::read_only);

    SharedData* data = static_cast<SharedData*>(region.get_address());

    std::cout << "Counter: " << data->counter << std::endl;
    std::cout << "Value: " << data->value << std::endl;
    std::cout << "Message: " << data->message << std::endl;

    // 清理
    bip::shared_memory_object::remove("SharedObject");
}
```

---

## 消息队列

```cpp
#include <boost/interprocess/ipc/message_queue.hpp>
#include <iostream>
#include <string>

namespace bip = boost::interprocess;

// 发送进程
void sender() {
    try {
        // 移除之前可能存在的队列
        bip::message_queue::remove("MessageQueue");

        // 创建消息队列
        bip::message_queue mq(
            bip::create_only,
            "MessageQueue",
            100,     // 最多 100 条消息
            256      // 每条消息最大 256 字节
        );

        // 发送消息
        std::string msg1 = "Hello";
        std::string msg2 = "World";
        std::string msg3 = "from message queue!";

        mq.send(msg1.data(), msg1.size(), 0);
        mq.send(msg2.data(), msg2.size(), 0);
        mq.send(msg3.data(), msg3.size(), 0);

        std::cout << "发送了 3 条消息" << std::endl;

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
}

// 接收进程
void receiver() {
    try {
        // 打开消息队列
        bip::message_queue mq(
            bip::open_only,
            "MessageQueue"
        );

        char buffer[256];
        std::size_t recvd_size;
        unsigned int priority;

        std::cout << "接收消息:\n";

        // 接收所有消息
        for (int i = 0; i < 3; ++i) {
            mq.receive(buffer, sizeof(buffer), recvd_size, priority);
            buffer[recvd_size] = '\0';
            std::cout << "  [" << i + 1 << "] " << buffer << std::endl;
        }

        // 清理
        bip::message_queue::remove("MessageQueue");

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
}
```

---

## 命名互斥锁

```cpp
#include <boost/interprocess/sync/named_mutex.hpp>
#include <boost/interprocess/sync/scoped_lock.hpp>
#include <iostream>
#include <thread>
#include <chrono>

namespace bip = boost::interprocess;

void critical_section(int process_id) {
    try {
        // 创建或打开命名互斥锁
        bip::named_mutex mutex(bip::open_or_create, "MyMutex");

        std::cout << "进程 " << process_id << " 尝试获取锁..." << std::endl;

        // 获取锁
        bip::scoped_lock<bip::named_mutex> lock(mutex);

        std::cout << "进程 " << process_id << " 进入临界区" << std::endl;

        // 模拟工作
        std::this_thread::sleep_for(std::chrono::seconds(2));

        std::cout << "进程 " << process_id << " 离开临界区" << std::endl;

        // lock 析构时自动释放锁

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
}

int main() {
    // 模拟多个进程访问
    std::thread t1(critical_section, 1);
    std::thread t2(critical_section, 2);

    t1.join();
    t2.join();

    // 清理
    bip::named_mutex::remove("MyMutex");

    return 0;
}
```

---

## 命名信号量

```cpp
#include <boost/interprocess/sync/named_semaphore.hpp>
#include <iostream>
#include <thread>
#include <chrono>

namespace bip = boost::interprocess;

void worker(int id, int count) {
    try {
        // 打开信号量
        bip::named_semaphore sem(bip::open_only, "MySemaphore");

        for (int i = 0; i < count; ++i) {
            std::cout << "Worker " << id << " 等待资源..." << std::endl;

            // 等待信号量
            sem.wait();

            std::cout << "Worker " << id << " 获得资源，处理中..." << std::endl;
            std::this_thread::sleep_for(std::chrono::seconds(1));

            // 释放信号量
            sem.post();

            std::cout << "Worker " << id << " 释放资源" << std::endl;
        }

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
}

int main() {
    try {
        // 移除旧的信号量
        bip::named_semaphore::remove("MySemaphore");

        // 创建信号量（初始计数为 2，允许 2 个进程同时访问）
        bip::named_semaphore sem(bip::create_only, "MySemaphore", 2);

        // 启动多个工作线程
        std::thread t1(worker, 1, 3);
        std::thread t2(worker, 2, 3);
        std::thread t3(worker, 3, 3);

        t1.join();
        t2.join();
        t3.join();

        // 清理
        bip::named_semaphore::remove("MySemaphore");

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 托管共享内存

```cpp
#include <boost/interprocess/managed_shared_memory.hpp>
#include <boost/interprocess/allocators/allocator.hpp>
#include <boost/interprocess/containers/vector.hpp>
#include <iostream>

namespace bip = boost::interprocess;

int main() {
    try {
        // 移除旧的共享内存
        bip::shared_memory_object::remove("ManagedMemory");

        // 创建托管共享内存
        bip::managed_shared_memory segment(
            bip::create_only,
            "ManagedMemory",
            65536  // 64KB
        );

        // 在共享内存中分配整数
        int* shared_int = segment.construct<int>("MyInt")(42);

        // 在共享内存中分配数组
        int* shared_array = segment.construct<int>("MyArray")[10]();
        for (int i = 0; i < 10; ++i) {
            shared_array[i] = i * i;
        }

        // 使用共享内存分配器的向量
        typedef bip::allocator<int, bip::managed_shared_memory::segment_manager>
            ShmAllocator;
        typedef bip::vector<int, ShmAllocator> ShmVector;

        ShmVector* shared_vector = segment.construct<ShmVector>("MyVector")(
            segment.get_segment_manager()
        );

        shared_vector->push_back(1);
        shared_vector->push_back(2);
        shared_vector->push_back(3);

        std::cout << "在共享内存中创建了对象" << std::endl;
        std::cout << "共享整数: " << *shared_int << std::endl;
        std::cout << "共享数组第5个元素: " << shared_array[5] << std::endl;
        std::cout << "共享向量大小: " << shared_vector->size() << std::endl;

        // 查找对象
        auto found_int = segment.find<int>("MyInt");
        if (found_int.first) {
            std::cout << "找到共享整数: " << *found_int.first << std::endl;
        }

        // 清理
        segment.destroy<int>("MyInt");
        segment.destroy<int>("MyArray");
        segment.destroy<ShmVector>("MyVector");

        bip::shared_memory_object::remove("ManagedMemory");

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 文件映射

```cpp
#include <boost/interprocess/file_mapping.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>
#include <fstream>
#include <cstring>

namespace bip = boost::interprocess;

int main() {
    const char* filename = "test_mapping.dat";

    try {
        // 创建文件
        {
            std::ofstream file(filename, std::ios::binary);
            const char data[] = "Hello, File Mapping!";
            file.write(data, sizeof(data));
        }

        // 映射文件
        bip::file_mapping mapping(filename, bip::read_write);
        bip::mapped_region region(mapping, bip::read_write);

        // 访问映射区域
        char* addr = static_cast<char*>(region.get_address());
        std::cout << "读取: " << addr << std::endl;

        // 修改数据
        std::strcpy(addr, "Modified via mapping");

        // 同步到磁盘
        region.flush();

        std::cout << "文件已通过内存映射修改" << std::endl;

    } catch (const bip::interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Interprocess 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/interprocess.html)
- [Boost.Interprocess 快速入门](https://www.boost.org/doc/libs/1_90_0/doc/html/interprocess/quick_guide.html)

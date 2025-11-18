# Boost.Interprocess - 进程间通信库

## 概述

Boost.Interprocess 提供共享内存、消息队列、同步原语等进程间通信机制。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/interprocess/shared_memory_object.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>
#include <cstring>

int main() {
    using namespace boost::interprocess;
    
    // 创建共享内存
    shared_memory_object shm(
        create_only,
        "MySharedMemory",
        read_write
    );
    
    shm.truncate(1024);
    
    // 映射区域
    mapped_region region(shm, read_write);
    
    // 写入数据
    std::strcpy(static_cast<char*>(region.get_address()), "Hello!");
    
    std::cout << "写入共享内存: Hello!" << std::endl;
    
    // 清理
    shared_memory_object::remove("MySharedMemory");
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_system -lpthread -lrt`

---

## 共享内存

```cpp
#include <boost/interprocess/shared_memory_object.hpp>
#include <boost/interprocess/mapped_region.hpp>
#include <iostream>
#include <cstring>

int main() {
    using namespace boost::interprocess;
    
    try {
        // 写进程
        shared_memory_object shm(
            create_only,
            "SharedData",
            read_write
        );
        
        shm.truncate(sizeof(int) * 10);
        
        mapped_region region(shm, read_write);
        int* data = static_cast<int*>(region.get_address());
        
        for (int i = 0; i < 10; ++i) {
            data[i] = i * 10;
        }
        
        std::cout << "写入10个整数到共享内存" << std::endl;
        
        // 读进程（在另一个程序中）
        // shared_memory_object shm(open_only, "SharedData", read_only);
        // mapped_region region(shm, read_only);
        // const int* data = static_cast<const int*>(region.get_address());
        
        shared_memory_object::remove("SharedData");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 消息队列

```cpp
#include <boost/interprocess/ipc/message_queue.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::interprocess;
    
    try {
        // 创建消息队列
        message_queue mq(
            create_only,
            "MessageQueue",
            100,      // 最大消息数
            sizeof(int)  // 每条消息大小
        );
        
        // 发送消息
        for (int i = 0; i < 5; ++i) {
            mq.send(&i, sizeof(i), 0);
            std::cout << "发送: " << i << std::endl;
        }
        
        // 接收消息（在另一个程序中）
        // message_queue mq(open_only, "MessageQueue");
        // int value;
        // unsigned int priority;
        // size_t received_size;
        // mq.receive(&value, sizeof(value), received_size, priority);
        
        message_queue::remove("MessageQueue");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 互斥锁

```cpp
#include <boost/interprocess/sync/named_mutex.hpp>
#include <boost/interprocess/sync/scoped_lock.hpp>
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    using namespace boost::interprocess;
    
    try {
        named_mutex mutex(open_or_create, "MyMutex");
        
        {
            scoped_lock<named_mutex> lock(mutex);
            std::cout << "进入临界区" << std::endl;
            std::this_thread::sleep_for(std::chrono::seconds(2));
            std::cout << "离开临界区" << std::endl;
        }
        
        named_mutex::remove("MyMutex");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 条件变量

```cpp
#include <boost/interprocess/sync/named_mutex.hpp>
#include <boost/interprocess/sync/named_condition.hpp>
#include <boost/interprocess/sync/scoped_lock.hpp>
#include <iostream>

int main() {
    using namespace boost::interprocess;
    
    try {
        named_mutex mutex(open_or_create, "CondMutex");
        named_condition cond(open_or_create, "CondVar");
        
        // 生产者
        {
            scoped_lock<named_mutex> lock(mutex);
            std::cout << "生产者准备就绪" << std::endl;
            cond.notify_one();
        }
        
        // 消费者（在另一个程序中）
        // scoped_lock<named_mutex> lock(mutex);
        // cond.wait(lock);
        // std::cout << "消费者收到通知" << std::endl;
        
        named_mutex::remove("CondMutex");
        named_condition::remove("CondVar");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 共享内存容器

```cpp
#include <boost/interprocess/managed_shared_memory.hpp>
#include <boost/interprocess/containers/vector.hpp>
#include <boost/interprocess/allocators/allocator.hpp>
#include <iostream>

int main() {
    using namespace boost::interprocess;
    
    typedef allocator<int, managed_shared_memory::segment_manager> ShmemAllocator;
    typedef vector<int, ShmemAllocator> ShmemVector;
    
    try {
        // 创建共享内存段
        managed_shared_memory segment(
            create_only,
            "SharedVector",
            65536
        );
        
        // 在共享内存中创建vector
        ShmemVector* vec = segment.construct<ShmemVector>("MyVector")(
            segment.get_segment_manager()
        );
        
        for (int i = 0; i < 10; ++i) {
            vec->push_back(i * 10);
        }
        
        std::cout << "共享内存 vector 大小: " << vec->size() << std::endl;
        
        shared_memory_object::remove("SharedVector");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 共享内存字符串

```cpp
#include <boost/interprocess/managed_shared_memory.hpp>
#include <boost/interprocess/containers/string.hpp>
#include <boost/interprocess/allocators/allocator.hpp>
#include <iostream>

int main() {
    using namespace boost::interprocess;
    
    typedef allocator<char, managed_shared_memory::segment_manager> CharAllocator;
    typedef basic_string<char, std::char_traits<char>, CharAllocator> ShmString;
    
    try {
        managed_shared_memory segment(
            create_only,
            "SharedString",
            65536
        );
        
        CharAllocator alloc(segment.get_segment_manager());
        ShmString* str = segment.construct<ShmString>("MyString")(alloc);
        
        *str = "Hello, Shared Memory!";
        
        std::cout << "共享字符串: " << *str << std::endl;
        
        shared_memory_object::remove("SharedString");
    }
    catch (const interprocess_exception& e) {
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
#include <fstream>
#include <iostream>
#include <cstring>

int main() {
    using namespace boost::interprocess;
    
    // 创建文件
    std::ofstream file("test.dat", std::ios::binary | std::ios::trunc);
    file.seekp(1023);
    file.write("", 1);
    file.close();
    
    try {
        // 映射文件
        file_mapping mapping("test.dat", read_write);
        mapped_region region(mapping, read_write);
        
        // 写入数据
        std::strcpy(static_cast<char*>(region.get_address()), 
                   "File-mapped data");
        
        std::cout << "写入文件映射: File-mapped data" << std::endl;
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 内存池

```cpp
#include <boost/interprocess/managed_shared_memory.hpp>
#include <iostream>

int main() {
    using namespace boost::interprocess;
    
    try {
        managed_shared_memory segment(
            create_only,
            "MemoryPool",
            65536
        );
        
        // 分配内存
        void* ptr1 = segment.allocate(100);
        void* ptr2 = segment.allocate(200);
        
        std::cout << "分配了两块内存" << std::endl;
        std::cout << "剩余空间: " << segment.get_free_memory() << " 字节" << std::endl;
        
        // 释放内存
        segment.deallocate(ptr1);
        segment.deallocate(ptr2);
        
        std::cout << "释放后剩余空间: " << segment.get_free_memory() << " 字节" << std::endl;
        
        shared_memory_object::remove("MemoryPool");
    }
    catch (const interprocess_exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Interprocess 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/interprocess.html)
- [进程间通信教程](https://www.boost.org/doc/libs/1_90_0/doc/html/interprocess/quick_guide.html)

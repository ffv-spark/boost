# Boost.Asio - 异步 I/O 库

## 概述

Boost.Asio 是一个跨平台的 C++ 库，用于网络和底层 I/O 编程，提供了一致的异步模型。

**类型**: 大部分为头文件库，部分功能需要链接

**链接库**: `-lboost_system -lpthread`

**主要特性**:
- 异步和同步 I/O 操作
- TCP/UDP socket 编程
- 定时器
- 串口通信
- SSL/TLS 支持
- HTTP 客户端/服务器（配合 Boost.Beast）

---

## 快速开始

```cpp
#include <boost/asio.hpp>
#include <iostream>

int main() {
    // 创建 I/O 上下文
    boost::asio::io_context io;

    // 创建定时器
    boost::asio::steady_timer timer(io, boost::asio::chrono::seconds(3));

    // 异步等待
    timer.async_wait([](const boost::system::error_code& ec) {
        if (!ec) {
            std::cout << "定时器触发！" << std::endl;
        }
    });

    // 运行事件循环
    io.run();

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_system -lpthread -o example
```

---

## 核心概念

### io_context

`io_context` 是 Asio 的核心，负责调度异步操作。

```cpp
#include <boost/asio.hpp>
#include <iostream>

int main() {
    boost::asio::io_context io;

    // 投递工作到 io_context
    io.post([]() {
        std::cout << "任务 1" << std::endl;
    });

    io.post([]() {
        std::cout << "任务 2" << std::endl;
    });

    // 运行所有任务
    io.run();

    return 0;
}
```

### 定时器

```cpp
#include <boost/asio.hpp>
#include <iostream>

namespace asio = boost::asio;

int main() {
    asio::io_context io;

    // 1. 同步定时器
    asio::steady_timer sync_timer(io, asio::chrono::seconds(1));
    sync_timer.wait();
    std::cout << "1 秒后..." << std::endl;

    // 2. 异步定时器
    asio::steady_timer async_timer(io, asio::chrono::seconds(2));
    async_timer.async_wait([](const boost::system::error_code& ec) {
        if (!ec) {
            std::cout << "2 秒后..." << std::endl;
        }
    });

    io.run();

    return 0;
}
```

### 重复定时器

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <memory>

class RepeatTimer {
public:
    RepeatTimer(boost::asio::io_context& io, int interval_sec, int count)
        : timer_(io, boost::asio::chrono::seconds(interval_sec))
        , interval_(interval_sec)
        , remaining_(count) {
        start();
    }

private:
    void start() {
        if (remaining_ > 0) {
            timer_.async_wait([this](const boost::system::error_code& ec) {
                if (!ec) {
                    std::cout << "触发 " << (--remaining_) << " 次剩余" << std::endl;

                    // 重新设置定时器
                    timer_.expires_after(boost::asio::chrono::seconds(interval_));
                    start();
                }
            });
        }
    }

    boost::asio::steady_timer timer_;
    int interval_;
    int remaining_;
};

int main() {
    boost::asio::io_context io;

    RepeatTimer timer(io, 1, 5); // 每秒触发，共 5 次

    io.run();

    return 0;
}
```

---

## TCP 网络编程

### TCP 服务器（同步）

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace asio = boost::asio;
using asio::ip::tcp;

int main() {
    try {
        asio::io_context io;

        // 创建接收器，监听端口 8080
        tcp::acceptor acceptor(io, tcp::endpoint(tcp::v4(), 8080));

        std::cout << "服务器启动，监听端口 8080..." << std::endl;

        while (true) {
            // 接受连接
            tcp::socket socket(io);
            acceptor.accept(socket);

            std::cout << "客户端连接: "
                      << socket.remote_endpoint().address().to_string()
                      << ":" << socket.remote_endpoint().port()
                      << std::endl;

            // 读取数据
            char data[1024];
            boost::system::error_code ec;
            size_t len = socket.read_some(asio::buffer(data), ec);

            if (!ec) {
                std::cout << "收到: " << std::string(data, len) << std::endl;

                // 发送响应
                std::string response = "HTTP/1.1 200 OK\r\n\r\nHello from Boost.Asio!";
                asio::write(socket, asio::buffer(response));
            }
        }

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

### TCP 服务器（异步）

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <memory>

namespace asio = boost::asio;
using asio::ip::tcp;

class Session : public std::enable_shared_from_this<Session> {
public:
    Session(tcp::socket socket) : socket_(std::move(socket)) {}

    void start() {
        do_read();
    }

private:
    void do_read() {
        auto self(shared_from_this());
        socket_.async_read_some(
            asio::buffer(data_, max_length),
            [this, self](boost::system::error_code ec, std::size_t length) {
                if (!ec) {
                    std::cout << "收到 " << length << " 字节: "
                              << std::string(data_, length) << std::endl;
                    do_write(length);
                }
            }
        );
    }

    void do_write(std::size_t length) {
        auto self(shared_from_this());
        asio::async_write(
            socket_,
            asio::buffer(data_, length),
            [this, self](boost::system::error_code ec, std::size_t /*length*/) {
                if (!ec) {
                    do_read();
                }
            }
        );
    }

    tcp::socket socket_;
    enum { max_length = 1024 };
    char data_[max_length];
};

class Server {
public:
    Server(asio::io_context& io, short port)
        : acceptor_(io, tcp::endpoint(tcp::v4(), port)) {
        do_accept();
    }

private:
    void do_accept() {
        acceptor_.async_accept(
            [this](boost::system::error_code ec, tcp::socket socket) {
                if (!ec) {
                    std::cout << "新连接" << std::endl;
                    std::make_shared<Session>(std::move(socket))->start();
                }
                do_accept();
            }
        );
    }

    tcp::acceptor acceptor_;
};

int main() {
    try {
        asio::io_context io;
        Server server(io, 8080);

        std::cout << "异步服务器启动，监听端口 8080..." << std::endl;

        io.run();

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

### TCP 客户端

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace asio = boost::asio;
using asio::ip::tcp;

int main() {
    try {
        asio::io_context io;

        // 解析主机和端口
        tcp::resolver resolver(io);
        tcp::resolver::results_type endpoints =
            resolver.resolve("www.example.com", "80");

        // 连接到服务器
        tcp::socket socket(io);
        asio::connect(socket, endpoints);

        std::cout << "已连接到服务器" << std::endl;

        // 发送 HTTP 请求
        std::string request =
            "GET / HTTP/1.1\r\n"
            "Host: www.example.com\r\n"
            "Connection: close\r\n\r\n";

        asio::write(socket, asio::buffer(request));

        // 读取响应
        std::string response;
        boost::system::error_code ec;

        while (true) {
            char buf[1024];
            size_t len = socket.read_some(asio::buffer(buf), ec);

            if (ec == asio::error::eof) {
                break; // 连接关闭
            } else if (ec) {
                throw boost::system::system_error(ec);
            }

            response.append(buf, len);
        }

        std::cout << "响应:\n" << response << std::endl;

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## UDP 网络编程

### UDP 服务器

```cpp
#include <boost/asio.hpp>
#include <iostream>

namespace asio = boost::asio;
using asio::ip::udp;

int main() {
    try {
        asio::io_context io;

        // 创建 UDP socket，绑定到端口 8080
        udp::socket socket(io, udp::endpoint(udp::v4(), 8080));

        std::cout << "UDP 服务器启动，监听端口 8080..." << std::endl;

        while (true) {
            char data[1024];
            udp::endpoint remote_endpoint;

            // 接收数据
            size_t len = socket.receive_from(
                asio::buffer(data), remote_endpoint);

            std::cout << "收到来自 "
                      << remote_endpoint.address().to_string()
                      << ":" << remote_endpoint.port()
                      << " 的数据: " << std::string(data, len)
                      << std::endl;

            // 发送响应
            std::string response = "收到: " + std::string(data, len);
            socket.send_to(asio::buffer(response), remote_endpoint);
        }

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

### UDP 客户端

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace asio = boost::asio;
using asio::ip::udp;

int main() {
    try {
        asio::io_context io;

        udp::socket socket(io, udp::v4());

        // 解析服务器地址
        udp::resolver resolver(io);
        udp::resolver::results_type endpoints =
            resolver.resolve(udp::v4(), "localhost", "8080");

        // 发送数据
        std::string message = "Hello, UDP server!";
        socket.send_to(asio::buffer(message), *endpoints.begin());

        // 接收响应
        char reply[1024];
        udp::endpoint sender_endpoint;
        size_t len = socket.receive_from(
            asio::buffer(reply), sender_endpoint);

        std::cout << "响应: " << std::string(reply, len) << std::endl;

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 异步操作模式

### 回调方式

```cpp
#include <boost/asio.hpp>
#include <iostream>

void async_operation(boost::asio::io_context& io) {
    auto timer = std::make_shared<boost::asio::steady_timer>(
        io, boost::asio::chrono::seconds(1));

    timer->async_wait([timer](const boost::system::error_code& ec) {
        if (!ec) {
            std::cout << "定时器触发" << std::endl;
        }
    });
}

int main() {
    boost::asio::io_context io;
    async_operation(io);
    io.run();
    return 0;
}
```

### 协程方式（C++20）

```cpp
#include <boost/asio.hpp>
#include <boost/asio/co_spawn.hpp>
#include <boost/asio/detached.hpp>
#include <boost/asio/use_awaitable.hpp>
#include <iostream>

namespace asio = boost::asio;

asio::awaitable<void> async_task() {
    auto executor = co_await asio::this_coro::executor;
    asio::steady_timer timer(executor, asio::chrono::seconds(1));

    co_await timer.async_wait(asio::use_awaitable);
    std::cout << "协程：定时器触发" << std::endl;
}

int main() {
    asio::io_context io;

    asio::co_spawn(io, async_task(), asio::detached);

    io.run();

    return 0;
}
```

---

## 多线程支持

### 线程池

```cpp
#include <boost/asio.hpp>
#include <boost/asio/thread_pool.hpp>
#include <iostream>
#include <vector>

int main() {
    // 创建线程池（4 个线程）
    boost::asio::thread_pool pool(4);

    // 提交任务
    for (int i = 0; i < 10; ++i) {
        boost::asio::post(pool, [i]() {
            std::cout << "任务 " << i
                      << " 在线程 " << std::this_thread::get_id()
                      << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        });
    }

    // 等待所有任务完成
    pool.join();

    return 0;
}
```

### 多线程 io_context

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    boost::asio::io_context io;

    // 防止 io_context 提前退出
    auto work = boost::asio::make_work_guard(io);

    // 创建多个线程运行 io_context
    std::vector<std::thread> threads;
    for (int i = 0; i < 4; ++i) {
        threads.emplace_back([&io, i]() {
            std::cout << "线程 " << i << " 启动" << std::endl;
            io.run();
            std::cout << "线程 " << i << " 退出" << std::endl;
        });
    }

    // 提交任务
    for (int i = 0; i < 10; ++i) {
        io.post([i]() {
            std::cout << "任务 " << i << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        });
    }

    // 等待一段时间后停止
    std::this_thread::sleep_for(std::chrono::seconds(2));
    work.reset();

    // 等待所有线程退出
    for (auto& t : threads) {
        t.join();
    }

    return 0;
}
```

---

## 串口通信

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace asio = boost::asio;

int main() {
    try {
        asio::io_context io;

        // 打开串口
        asio::serial_port port(io, "/dev/ttyUSB0");

        // 配置串口
        port.set_option(asio::serial_port_base::baud_rate(9600));
        port.set_option(asio::serial_port_base::character_size(8));
        port.set_option(asio::serial_port_base::parity(
            asio::serial_port_base::parity::none));
        port.set_option(asio::serial_port_base::stop_bits(
            asio::serial_port_base::stop_bits::one));

        // 发送数据
        std::string message = "Hello, Serial!\n";
        asio::write(port, asio::buffer(message));

        // 读取数据
        char data[128];
        size_t len = port.read_some(asio::buffer(data));

        std::cout << "收到: " << std::string(data, len) << std::endl;

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 实用示例

### HTTP 客户端

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace asio = boost::asio;
using asio::ip::tcp;

std::string http_get(const std::string& host, const std::string& path) {
    asio::io_context io;

    // 解析主机
    tcp::resolver resolver(io);
    auto endpoints = resolver.resolve(host, "80");

    // 连接
    tcp::socket socket(io);
    asio::connect(socket, endpoints);

    // 构造 HTTP 请求
    std::string request =
        "GET " + path + " HTTP/1.1\r\n"
        "Host: " + host + "\r\n"
        "Connection: close\r\n\r\n";

    // 发送请求
    asio::write(socket, asio::buffer(request));

    // 读取响应
    std::string response;
    boost::system::error_code ec;

    while (true) {
        char buf[1024];
        size_t len = socket.read_some(asio::buffer(buf), ec);

        if (ec == asio::error::eof) break;
        if (ec) throw boost::system::system_error(ec);

        response.append(buf, len);
    }

    return response;
}

int main() {
    try {
        std::string response = http_get("www.example.com", "/");
        std::cout << response << std::endl;
    } catch (std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

### 聊天服务器

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <memory>
#include <set>
#include <deque>

namespace asio = boost::asio;
using asio::ip::tcp;

using message_queue = std::deque<std::string>;

class ChatRoom {
public:
    void join(std::shared_ptr<class Participant> participant);
    void leave(std::shared_ptr<class Participant> participant);
    void deliver(const std::string& msg);

private:
    std::set<std::shared_ptr<class Participant>> participants_;
    enum { max_recent_msgs = 100 };
    message_queue recent_msgs_;
};

class Participant {
public:
    virtual ~Participant() {}
    virtual void deliver(const std::string& msg) = 0;
};

// ... 完整实现略，见官方示例
```

---

## 最佳实践

1. **异步优先**: 对于 I/O 密集型应用，使用异步操作
2. **共享资源保护**: 多线程环境下使用 strand 保护共享资源
3. **错误处理**: 始终检查 error_code
4. **资源管理**: 使用智能指针管理异步对象生命周期
5. **超时处理**: 为网络操作设置超时

---

## 参考资源

- [Boost.Asio 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_asio.html)
- [Boost.Asio 示例](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_asio/examples.html)
- [Boost.Beast](https://www.boost.org/doc/libs/1_90_0/libs/beast/doc/html/index.html) - HTTP/WebSocket

# Boost.Asio - 异步IO库

## 概述

Boost.Asio 提供异步I/O和网络编程功能，支持TCP、UDP、串口等。

**类型**: 仅头文件库（某些功能需要链接）

---

## 快速开始

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

int main() {
    boost::asio::io_context io;

    // 创建定时器
    boost::asio::steady_timer timer(io, boost::asio::chrono::seconds(2));

    timer.async_wait([](const boost::system::error_code& ec) {
        if (!ec) {
            std::cout << "定时器触发!" << std::endl;
        }
    });

    std::cout << "等待定时器..." << std::endl;
    io.run();

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_system -lpthread`

---

## TCP 服务器

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <memory>

using boost::asio::ip::tcp;

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
            boost::asio::buffer(data_, max_length),
            [this, self](boost::system::error_code ec, std::size_t length) {
                if (!ec) {
                    std::cout << "收到: " << std::string(data_, length) << std::endl;
                    do_write(length);
                }
            });
    }

    void do_write(std::size_t length) {
        auto self(shared_from_this());
        boost::asio::async_write(
            socket_, boost::asio::buffer(data_, length),
            [this, self](boost::system::error_code ec, std::size_t /*length*/) {
                if (!ec) {
                    do_read();
                }
            });
    }

    tcp::socket socket_;
    enum { max_length = 1024 };
    char data_[max_length];
};

class Server {
public:
    Server(boost::asio::io_context& io, short port)
        : acceptor_(io, tcp::endpoint(tcp::v4(), port)) {
        do_accept();
    }

private:
    void do_accept() {
        acceptor_.async_accept(
            [this](boost::system::error_code ec, tcp::socket socket) {
                if (!ec) {
                    std::make_shared<Session>(std::move(socket))->start();
                }
                do_accept();
            });
    }

    tcp::acceptor acceptor_;
};

int main() {
    try {
        boost::asio::io_context io;
        Server server(io, 8080);
        std::cout << "服务器监听端口 8080..." << std::endl;
        io.run();
    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## TCP 客户端

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

using boost::asio::ip::tcp;

int main() {
    try {
        boost::asio::io_context io;

        tcp::resolver resolver(io);
        auto endpoints = resolver.resolve("localhost", "8080");

        tcp::socket socket(io);
        boost::asio::connect(socket, endpoints);

        std::cout << "已连接到服务器" << std::endl;

        // 发送消息
        std::string message = "Hello, Server!";
        boost::asio::write(socket, boost::asio::buffer(message));

        // 接收回复
        char reply[1024];
        size_t reply_length = boost::asio::read(
            socket, boost::asio::buffer(reply, message.length()));

        std::cout << "回复: " << std::string(reply, reply_length) << std::endl;

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## HTTP 客户端

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <string>

using boost::asio::ip::tcp;

int main() {
    try {
        boost::asio::io_context io;

        tcp::resolver resolver(io);
        auto endpoints = resolver.resolve("www.example.com", "80");

        tcp::socket socket(io);
        boost::asio::connect(socket, endpoints);

        // 发送 HTTP 请求
        std::string request = 
            "GET / HTTP/1.1\r\n"
            "Host: www.example.com\r\n"
            "Connection: close\r\n\r\n";

        boost::asio::write(socket, boost::asio::buffer(request));

        // 读取响应
        boost::asio::streambuf response;
        boost::asio::read_until(socket, response, "\r\n");

        std::istream response_stream(&response);
        std::string http_version;
        unsigned int status_code;
        std::string status_message;

        response_stream >> http_version >> status_code;
        std::getline(response_stream, status_message);

        std::cout << "HTTP/" << http_version << " " 
                  << status_code << " " << status_message << std::endl;

    } catch (std::exception& e) {
        std::cerr << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 定时器

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <chrono>

int main() {
    boost::asio::io_context io;

    // 一次性定时器
    boost::asio::steady_timer timer1(io, std::chrono::seconds(1));
    timer1.async_wait([](const boost::system::error_code& ec) {
        std::cout << "1秒定时器触发" << std::endl;
    });

    // 周期性定时器
    auto timer2 = std::make_shared<boost::asio::steady_timer>(io);
    std::function<void(const boost::system::error_code&)> periodic_handler;

    periodic_handler = [timer2, &periodic_handler](const boost::system::error_code& ec) {
        if (!ec) {
            std::cout << "周期性定时器触发" << std::endl;
            timer2->expires_after(std::chrono::seconds(1));
            timer2->async_wait(periodic_handler);
        }
    };

    timer2->expires_after(std::chrono::seconds(2));
    timer2->async_wait(periodic_handler);

    io.run();

    return 0;
}
```

---

## UDP 通信

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <array>

using boost::asio::ip::udp;

// UDP 服务器
void udp_server() {
    boost::asio::io_context io;
    udp::socket socket(io, udp::endpoint(udp::v4(), 8080));

    std::array<char, 1024> recv_buf;
    udp::endpoint remote_endpoint;

    socket.async_receive_from(
        boost::asio::buffer(recv_buf), remote_endpoint,
        [&](boost::system::error_code ec, std::size_t bytes_recvd) {
            if (!ec && bytes_recvd > 0) {
                std::cout << "收到: " << std::string(recv_buf.data(), bytes_recvd) << std::endl;

                // 回复
                socket.send_to(
                    boost::asio::buffer(recv_buf, bytes_recvd),
                    remote_endpoint);
            }
        });

    io.run();
}

// UDP 客户端
void udp_client() {
    boost::asio::io_context io;
    udp::socket socket(io, udp::endpoint(udp::v4(), 0));

    udp::resolver resolver(io);
    udp::endpoint receiver_endpoint = 
        *resolver.resolve(udp::v4(), "localhost", "8080").begin();

    std::string message = "Hello UDP!";
    socket.send_to(boost::asio::buffer(message), receiver_endpoint);

    std::array<char, 1024> recv_buf;
    udp::endpoint sender_endpoint;
    size_t len = socket.receive_from(
        boost::asio::buffer(recv_buf), sender_endpoint);

    std::cout << "回复: " << std::string(recv_buf.data(), len) << std::endl;
}

int main() {
    // udp_server();
    udp_client();
    return 0;
}
```

---

## 异步读写文件

```cpp
#include <boost/asio.hpp>
#include <boost/asio/posix/stream_descriptor.hpp>
#include <iostream>
#include <fcntl.h>
#include <unistd.h>

int main() {
    boost::asio::io_context io;

    // 创建文件
    int fd = open("test.txt", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        std::cerr << "无法打开文件" << std::endl;
        return 1;
    }

    boost::asio::posix::stream_descriptor stream(io, fd);

    std::string data = "Hello, Async File I/O!";

    // 异步写入
    boost::asio::async_write(
        stream, boost::asio::buffer(data),
        [](boost::system::error_code ec, std::size_t bytes_written) {
            if (!ec) {
                std::cout << "写入 " << bytes_written << " 字节" << std::endl;
            }
        });

    io.run();

    return 0;
}
```

---

## 协程支持

```cpp
#include <boost/asio.hpp>
#include <boost/asio/spawn.hpp>
#include <iostream>

using boost::asio::ip::tcp;

void session(boost::asio::yield_context yield, tcp::socket socket) {
    try {
        char data[1024];
        for (;;) {
            std::size_t n = socket.async_read_some(
                boost::asio::buffer(data), yield);

            boost::asio::async_write(socket,
                boost::asio::buffer(data, n), yield);
        }
    } catch (std::exception& e) {
        std::cout << "会话结束: " << e.what() << std::endl;
    }
}

void server(boost::asio::io_context& io, short port) {
    boost::asio::spawn(io, [&io, port](boost::asio::yield_context yield) {
        tcp::acceptor acceptor(io, tcp::endpoint(tcp::v4(), port));

        for (;;) {
            boost::system::error_code ec;
            tcp::socket socket(io);
            acceptor.async_accept(socket, yield[ec]);

            if (!ec) {
                boost::asio::spawn(io,
                    [s = std::move(socket)](boost::asio::yield_context yield) mutable {
                        session(yield, std::move(s));
                    });
            }
        }
    });
}

int main() {
    boost::asio::io_context io;
    server(io, 8080);
    io.run();
    return 0;
}
```

**需要**: `g++ -std=c++14 example.cpp -lboost_coroutine -lboost_context -lboost_system -lpthread`

---

## 信号处理

```cpp
#include <boost/asio.hpp>
#include <iostream>
#include <csignal>

int main() {
    boost::asio::io_context io;

    boost::asio::signal_set signals(io, SIGINT, SIGTERM);

    signals.async_wait([&](const boost::system::error_code& ec, int signal_number) {
        if (!ec) {
            std::cout << "\n收到信号 " << signal_number << "，正在退出..." << std::endl;
            io.stop();
        }
    });

    std::cout << "按 Ctrl+C 退出..." << std::endl;
    io.run();

    return 0;
}
```

---

## 参考资源

- [Boost.Asio 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_asio.html)
- [Asio 示例](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_asio/examples.html)

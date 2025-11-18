# Boost.Beast - HTTP 和 WebSocket 库

## 概述

Boost.Beast 是一个基于 Boost.Asio 的 HTTP 和 WebSocket 库，提供了底层的网络协议支持。

**类型**: 仅头文件库（大部分）

**依赖**: Boost.Asio, Boost.System

**链接库**: `-lboost_system -lpthread`

**主要特性**:
- HTTP/1.1 客户端和服务器
- WebSocket 客户端和服务器
- 同步和异步操作
- 零拷贝设计
- 完全兼容标准

---

## 快速开始

### HTTP 客户端

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/connect.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <iostream>
#include <string>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;

int main() {
    try {
        net::io_context ioc;

        // 解析主机
        tcp::resolver resolver(ioc);
        auto results = resolver.resolve("www.example.com", "80");

        // 连接
        beast::tcp_stream stream(ioc);
        stream.connect(results);

        // 构造 HTTP GET 请求
        http::request<http::string_body> req{http::verb::get, "/", 11};
        req.set(http::field::host, "www.example.com");
        req.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);

        // 发送请求
        http::write(stream, req);

        // 读取响应
        beast::flat_buffer buffer;
        http::response<http::dynamic_body> res;
        http::read(stream, buffer, res);

        // 输出响应
        std::cout << res << std::endl;

        // 关闭连接
        beast::error_code ec;
        stream.socket().shutdown(tcp::socket::shutdown_both, ec);

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

**编译**:
```bash
g++ -std=c++14 example.cpp -lboost_system -lpthread -o example
```

---

## HTTP 服务器

### 同步 HTTP 服务器

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <iostream>
#include <string>
#include <thread>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;

// 处理 HTTP 请求
template<class Body, class Allocator>
http::message_generator
handle_request(http::request<Body, http::basic_fields<Allocator>>&& req) {
    // 返回 404
    auto const not_found = [&req](beast::string_view target) {
        http::response<http::string_body> res{http::status::not_found, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = "The resource '" + std::string(target) + "' was not found.";
        res.prepare_payload();
        return res;
    };

    // 根据路径响应
    if (req.target() == "/") {
        http::response<http::string_body> res{http::status::ok, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = "<html><body><h1>Hello, Beast!</h1></body></html>";
        res.prepare_payload();
        return res;
    }

    if (req.target() == "/api/info") {
        http::response<http::string_body> res{http::status::ok, req.version()};
        res.set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res.set(http::field::content_type, "application/json");
        res.keep_alive(req.keep_alive());
        res.body() = R"({"name":"Beast Server","version":"1.0"})";
        res.prepare_payload();
        return res;
    }

    return not_found(req.target());
}

// 处理单个会话
void do_session(tcp::socket socket) {
    try {
        beast::tcp_stream stream(std::move(socket));

        beast::flat_buffer buffer;

        for(;;) {
            // 读取请求
            http::request<http::string_body> req;
            http::read(stream, buffer, req);

            // 处理请求
            http::message_generator msg = handle_request(std::move(req));

            // 发送响应
            bool keep_alive = msg.keep_alive();
            beast::write(stream, std::move(msg));

            if (!keep_alive) {
                break;
            }
        }

        stream.socket().shutdown(tcp::socket::shutdown_send);

    } catch (beast::system_error const& se) {
        if (se.code() != http::error::end_of_stream)
            std::cerr << "错误: " << se.code().message() << std::endl;
    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }
}

int main() {
    try {
        auto const address = net::ip::make_address("0.0.0.0");
        unsigned short port = 8080;

        net::io_context ioc{1};

        tcp::acceptor acceptor{ioc, {address, port}};

        std::cout << "HTTP 服务器运行在 http://0.0.0.0:" << port << std::endl;

        for(;;) {
            tcp::socket socket{ioc};
            acceptor.accept(socket);

            std::thread{std::bind(&do_session, std::move(socket))}.detach();
        }

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }
}
```

### 异步 HTTP 服务器

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio.hpp>
#include <memory>
#include <iostream>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;

class HttpSession : public std::enable_shared_from_this<HttpSession> {
    beast::tcp_stream stream_;
    beast::flat_buffer buffer_;
    http::request<http::string_body> req_;

public:
    HttpSession(tcp::socket&& socket)
        : stream_(std::move(socket)) {}

    void run() {
        do_read();
    }

private:
    void do_read() {
        auto self = shared_from_this();

        http::async_read(stream_, buffer_, req_,
            [self](beast::error_code ec, std::size_t bytes_transferred) {
                boost::ignore_unused(bytes_transferred);

                if (ec == http::error::end_of_stream) {
                    self->do_close();
                    return;
                }

                if (ec) {
                    std::cerr << "读取错误: " << ec.message() << std::endl;
                    return;
                }

                self->handle_request();
            });
    }

    void handle_request() {
        auto self = shared_from_this();

        // 创建响应
        auto res = std::make_shared<http::response<http::string_body>>(
            http::status::ok, req_.version());

        res->set(http::field::server, BOOST_BEAST_VERSION_STRING);
        res->set(http::field::content_type, "text/plain");
        res->keep_alive(req_.keep_alive());
        res->body() = "Hello from async Beast server!";
        res->prepare_payload();

        http::async_write(stream_, *res,
            [self, res](beast::error_code ec, std::size_t) {
                if (ec) {
                    std::cerr << "写入错误: " << ec.message() << std::endl;
                    return;
                }

                if (!res->keep_alive()) {
                    self->do_close();
                    return;
                }

                self->do_read();
            });
    }

    void do_close() {
        beast::error_code ec;
        stream_.socket().shutdown(tcp::socket::shutdown_send, ec);
    }
};

class HttpServer {
    net::io_context& ioc_;
    tcp::acceptor acceptor_;

public:
    HttpServer(net::io_context& ioc, tcp::endpoint endpoint)
        : ioc_(ioc)
        , acceptor_(ioc) {
        beast::error_code ec;

        acceptor_.open(endpoint.protocol(), ec);
        acceptor_.set_option(net::socket_base::reuse_address(true), ec);
        acceptor_.bind(endpoint, ec);
        acceptor_.listen(net::socket_base::max_listen_connections, ec);

        do_accept();
    }

private:
    void do_accept() {
        acceptor_.async_accept(
            net::make_strand(ioc_),
            [this](beast::error_code ec, tcp::socket socket) {
                if (!ec) {
                    std::make_shared<HttpSession>(std::move(socket))->run();
                }

                do_accept();
            });
    }
};

int main() {
    try {
        auto const address = net::ip::make_address("0.0.0.0");
        unsigned short port = 8080;

        net::io_context ioc{4};  // 4 个线程

        HttpServer server{ioc, {address, port}};

        std::cout << "异步 HTTP 服务器运行在 http://0.0.0.0:" << port << std::endl;

        ioc.run();

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## WebSocket

### WebSocket 服务器

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/websocket.hpp>
#include <boost/asio.hpp>
#include <iostream>
#include <string>
#include <memory>

namespace beast = boost::beast;
namespace websocket = beast::websocket;
namespace net = boost::asio;
using tcp = net::ip::tcp;

class WebSocketSession : public std::enable_shared_from_this<WebSocketSession> {
    websocket::stream<beast::tcp_stream> ws_;
    beast::flat_buffer buffer_;

public:
    explicit WebSocketSession(tcp::socket&& socket)
        : ws_(std::move(socket)) {}

    void run() {
        // 设置选项
        ws_.set_option(websocket::stream_base::timeout::suggested(
            beast::role_type::server));

        ws_.set_option(websocket::stream_base::decorator(
            [](websocket::response_type& res) {
                res.set(beast::http::field::server, "Beast WebSocket Server");
            }));

        // 接受 WebSocket 握手
        ws_.async_accept(
            [self = shared_from_this()](beast::error_code ec) {
                if (ec) {
                    std::cerr << "接受错误: " << ec.message() << std::endl;
                    return;
                }

                self->do_read();
            });
    }

private:
    void do_read() {
        auto self = shared_from_this();

        ws_.async_read(buffer_,
            [self](beast::error_code ec, std::size_t bytes_transferred) {
                boost::ignore_unused(bytes_transferred);

                if (ec == websocket::error::closed) {
                    return;
                }

                if (ec) {
                    std::cerr << "读取错误: " << ec.message() << std::endl;
                    return;
                }

                // 回显消息
                self->ws_.text(self->ws_.got_text());
                self->do_write();
            });
    }

    void do_write() {
        auto self = shared_from_this();

        ws_.async_write(buffer_.data(),
            [self](beast::error_code ec, std::size_t bytes_transferred) {
                boost::ignore_unused(bytes_transferred);

                if (ec) {
                    std::cerr << "写入错误: " << ec.message() << std::endl;
                    return;
                }

                self->buffer_.consume(self->buffer_.size());
                self->do_read();
            });
    }
};

class WebSocketServer {
    net::io_context& ioc_;
    tcp::acceptor acceptor_;

public:
    WebSocketServer(net::io_context& ioc, tcp::endpoint endpoint)
        : ioc_(ioc)
        , acceptor_(ioc) {
        beast::error_code ec;

        acceptor_.open(endpoint.protocol(), ec);
        acceptor_.set_option(net::socket_base::reuse_address(true), ec);
        acceptor_.bind(endpoint, ec);
        acceptor_.listen(net::socket_base::max_listen_connections, ec);

        do_accept();
    }

private:
    void do_accept() {
        acceptor_.async_accept(
            net::make_strand(ioc_),
            [this](beast::error_code ec, tcp::socket socket) {
                if (!ec) {
                    std::make_shared<WebSocketSession>(std::move(socket))->run();
                }

                do_accept();
            });
    }
};

int main() {
    try {
        auto const address = net::ip::make_address("0.0.0.0");
        unsigned short port = 8080;

        net::io_context ioc{1};

        WebSocketServer server{ioc, {address, port}};

        std::cout << "WebSocket 服务器运行在 ws://0.0.0.0:" << port << std::endl;

        ioc.run();

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

### WebSocket 客户端

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/websocket.hpp>
#include <boost/asio.hpp>
#include <iostream>
#include <string>

namespace beast = boost::beast;
namespace websocket = beast::websocket;
namespace net = boost::asio;
using tcp = net::ip::tcp;

int main() {
    try {
        net::io_context ioc;

        // 解析主机
        tcp::resolver resolver{ioc};
        auto results = resolver.resolve("echo.websocket.org", "80");

        // 连接
        websocket::stream<tcp::socket> ws{ioc};
        auto ep = net::connect(ws.next_layer(), results);

        // 设置 Host 头
        std::string host = "echo.websocket.org:80";

        // WebSocket 握手
        ws.handshake(host, "/");

        // 发送消息
        std::string message = "Hello, WebSocket!";
        ws.write(net::buffer(message));

        // 读取响应
        beast::flat_buffer buffer;
        ws.read(buffer);

        std::cout << "收到: " << beast::make_printable(buffer.data()) << std::endl;

        // 关闭连接
        ws.close(websocket::close_code::normal);

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## REST API 服务器

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/asio.hpp>
#include <boost/json.hpp>
#include <iostream>
#include <map>
#include <string>
#include <memory>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
namespace json = boost::json;
using tcp = net::ip::tcp;

// 简单的内存数据库
class Database {
public:
    std::string get(int id) {
        auto it = data_.find(id);
        if (it != data_.end()) {
            return it->second;
        }
        return "";
    }

    void set(int id, const std::string& value) {
        data_[id] = value;
    }

    void remove(int id) {
        data_.erase(id);
    }

    json::array list() {
        json::array arr;
        for (const auto& [id, value] : data_) {
            json::object obj;
            obj["id"] = id;
            obj["value"] = value;
            arr.push_back(obj);
        }
        return arr;
    }

private:
    std::map<int, std::string> data_;
};

Database db;

// 路由处理
template<class Body, class Allocator>
http::message_generator
route_request(http::request<Body, http::basic_fields<Allocator>>&& req) {
    auto make_response = [&req](http::status status, const std::string& body,
                                 const std::string& content_type = "application/json") {
        http::response<http::string_body> res{status, req.version()};
        res.set(http::field::server, "Beast REST API");
        res.set(http::field::content_type, content_type);
        res.keep_alive(req.keep_alive());
        res.body() = body;
        res.prepare_payload();
        return res;
    };

    // GET /api/items - 列出所有项
    if (req.method() == http::verb::get && req.target() == "/api/items") {
        json::object response;
        response["items"] = db.list();
        return make_response(http::status::ok, json::serialize(response));
    }

    // GET /api/items/{id} - 获取单个项
    if (req.method() == http::verb::get && req.target().starts_with("/api/items/")) {
        std::string id_str = std::string(req.target().substr(11));
        int id = std::stoi(id_str);

        std::string value = db.get(id);
        if (!value.empty()) {
            json::object response;
            response["id"] = id;
            response["value"] = value;
            return make_response(http::status::ok, json::serialize(response));
        }

        json::object error;
        error["error"] = "Not found";
        return make_response(http::status::not_found, json::serialize(error));
    }

    // POST /api/items - 创建项
    if (req.method() == http::verb::post && req.target() == "/api/items") {
        auto body = json::parse(req.body());
        int id = body.at("id").as_int64();
        std::string value = body.at("value").as_string().c_str();

        db.set(id, value);

        json::object response;
        response["success"] = true;
        response["id"] = id;
        return make_response(http::status::created, json::serialize(response));
    }

    // DELETE /api/items/{id} - 删除项
    if (req.method() == http::verb::delete_ && req.target().starts_with("/api/items/")) {
        std::string id_str = std::string(req.target().substr(11));
        int id = std::stoi(id_str);

        db.remove(id);

        json::object response;
        response["success"] = true;
        return make_response(http::status::ok, json::serialize(response));
    }

    // 404
    json::object error;
    error["error"] = "Not found";
    return make_response(http::status::not_found, json::serialize(error));
}

// 省略会话和服务器类（与前面的异步服务器类似）
// ...

int main() {
    // 初始化一些数据
    db.set(1, "Apple");
    db.set(2, "Banana");
    db.set(3, "Cherry");

    std::cout << "REST API 服务器运行在 http://0.0.0.0:8080" << std::endl;
    std::cout << "端点:" << std::endl;
    std::cout << "  GET    /api/items" << std::endl;
    std::cout << "  GET    /api/items/{id}" << std::endl;
    std::cout << "  POST   /api/items" << std::endl;
    std::cout << "  DELETE /api/items/{id}" << std::endl;

    // 启动服务器...
    return 0;
}
```

---

## 文件服务器

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/asio.hpp>
#include <iostream>
#include <fstream>
#include <string>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;

// MIME 类型
std::string mime_type(const std::string& path) {
    if (path.ends_with(".html") || path.ends_with(".htm")) return "text/html";
    if (path.ends_with(".css")) return "text/css";
    if (path.ends_with(".js")) return "application/javascript";
    if (path.ends_with(".json")) return "application/json";
    if (path.ends_with(".png")) return "image/png";
    if (path.ends_with(".jpg") || path.ends_with(".jpeg")) return "image/jpeg";
    if (path.ends_with(".gif")) return "image/gif";
    if (path.ends_with(".txt")) return "text/plain";
    return "application/octet-stream";
}

// 提供文件
template<class Body, class Allocator>
http::message_generator
serve_file(http::request<Body, http::basic_fields<Allocator>>&& req,
           const std::string& doc_root) {
    // 构造完整路径
    std::string path = doc_root;
    std::string target(req.target());

    if (target == "/") {
        target = "/index.html";
    }

    path += target;

    // 读取文件
    std::ifstream file(path, std::ios::binary);
    if (!file) {
        http::response<http::string_body> res{http::status::not_found, req.version()};
        res.set(http::field::server, "Beast File Server");
        res.set(http::field::content_type, "text/html");
        res.keep_alive(req.keep_alive());
        res.body() = "<html><body><h1>404 Not Found</h1></body></html>";
        res.prepare_payload();
        return res;
    }

    // 读取文件内容
    std::string content((std::istreambuf_iterator<char>(file)),
                        std::istreambuf_iterator<char>());

    http::response<http::string_body> res{http::status::ok, req.version()};
    res.set(http::field::server, "Beast File Server");
    res.set(http::field::content_type, mime_type(path));
    res.keep_alive(req.keep_alive());
    res.body() = content;
    res.prepare_payload();
    return res;
}
```

---

## HTTPS 支持

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/ssl.hpp>
#include <boost/asio.hpp>
#include <boost/asio/ssl.hpp>
#include <iostream>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
namespace ssl = net::ssl;
using tcp = net::ip::tcp;

int main() {
    try {
        net::io_context ioc;

        // SSL 上下文
        ssl::context ctx{ssl::context::tlsv12_client};

        // 加载根证书
        ctx.set_default_verify_paths();

        // 解析主机
        tcp::resolver resolver{ioc};
        auto results = resolver.resolve("www.google.com", "443");

        // SSL 流
        beast::ssl_stream<beast::tcp_stream> stream{ioc, ctx};

        // SNI 主机名
        if (!SSL_set_tlsext_host_name(stream.native_handle(), "www.google.com")) {
            throw beast::system_error{
                beast::error_code{static_cast<int>(::ERR_get_error()),
                                  net::error::get_ssl_category()},
                "Failed to set SNI hostname"};
        }

        // 连接
        beast::get_lowest_layer(stream).connect(results);

        // SSL 握手
        stream.handshake(ssl::stream_base::client);

        // HTTP 请求
        http::request<http::string_body> req{http::verb::get, "/", 11};
        req.set(http::field::host, "www.google.com");
        req.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);

        http::write(stream, req);

        // 读取响应
        beast::flat_buffer buffer;
        http::response<http::dynamic_body> res;
        http::read(stream, buffer, res);

        std::cout << res << std::endl;

        // 关闭
        beast::error_code ec;
        stream.shutdown(ec);

    } catch (std::exception const& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## 最佳实践

1. **使用异步操作**: 提高性能和并发能力
2. **错误处理**: 正确处理所有错误情况
3. **资源管理**: 使用 RAII 和智能指针
4. **超时设置**: 设置合理的超时时间
5. **日志记录**: 记录重要事件和错误
6. **安全性**: 验证输入，防止攻击

---

## 编译选项

```bash
# 基本编译（HTTP）
g++ -std=c++14 example.cpp -lboost_system -lpthread -o example

# HTTPS（需要 OpenSSL）
g++ -std=c++14 https_example.cpp -lboost_system -lssl -lcrypto -lpthread -o https_example

# CMake
find_package(Boost REQUIRED COMPONENTS system)
find_package(OpenSSL REQUIRED)
target_link_libraries(myapp Boost::system OpenSSL::SSL OpenSSL::Crypto)
```

---

## 参考资源

- [Boost.Beast 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/beast/doc/html/index.html)
- [Beast 示例](https://www.boost.org/doc/libs/1_90_0/libs/beast/example/)
- [WebSocket RFC](https://tools.ietf.org/html/rfc6455)

# Boost.Beast - HTTP/WebSocket库

## 概述

Boost.Beast 提供HTTP和WebSocket协议实现，基于Boost.Asio构建。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/asio.hpp>
#include <iostream>

namespace beast = boost::beast;
namespace http = beast::http;
namespace net = boost::asio;
using tcp = net::ip::tcp;

int main() {
    // HTTP GET请求示例（需要实际的网络连接）
    std::cout << "Boost.Beast HTTP/WebSocket库" << std::endl;
    std::cout << "用于构建高性能的HTTP和WebSocket应用" << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Beast 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/beast/doc/html/index.html)

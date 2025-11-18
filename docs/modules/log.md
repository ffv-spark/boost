# Boost.Log - 日志库

## 概述

Boost.Log 提供强大、灵活、可扩展的日志记录系统，支持多种输出格式和过滤机制。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/log/trivial.hpp>

int main() {
    BOOST_LOG_TRIVIAL(trace) << "这是一条 trace 消息";
    BOOST_LOG_TRIVIAL(debug) << "这是一条 debug 消息";
    BOOST_LOG_TRIVIAL(info) << "这是一条 info 消息";
    BOOST_LOG_TRIVIAL(warning) << "这是一条 warning 消息";
    BOOST_LOG_TRIVIAL(error) << "这是一条 error 消息";
    BOOST_LOG_TRIVIAL(fatal) << "这是一条 fatal 消息";

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_log -lboost_log_setup -lboost_thread -lboost_system -lpthread -DBOOST_LOG_DYN_LINK`

---

## 基本日志记录

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/expressions.hpp>

namespace logging = boost::log;

void init_logging() {
    // 设置最小严重级别
    logging::core::get()->set_filter(
        logging::trivial::severity >= logging::trivial::info
    );
}

int main() {
    init_logging();

    BOOST_LOG_TRIVIAL(trace) << "不会显示";
    BOOST_LOG_TRIVIAL(debug) << "不会显示";
    BOOST_LOG_TRIVIAL(info) << "会显示";
    BOOST_LOG_TRIVIAL(warning) << "会显示";
    BOOST_LOG_TRIVIAL(error) << "会显示";

    return 0;
}
```

---

## 文件日志

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/utility/setup/file.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

namespace logging = boost::log;
namespace keywords = boost::log::keywords;

void init_file_logging() {
    logging::add_file_log(
        keywords::file_name = "app_%N.log",
        keywords::rotation_size = 10 * 1024 * 1024,  // 10 MB
        keywords::time_based_rotation = 
            logging::sinks::file::rotation_at_time_point(0, 0, 0),  // 每天午夜
        keywords::format = "[%TimeStamp%] [%Severity%] %Message%"
    );

    logging::add_common_attributes();
}

int main() {
    init_file_logging();

    BOOST_LOG_TRIVIAL(info) << "应用启动";
    BOOST_LOG_TRIVIAL(warning) << "这是一个警告";
    BOOST_LOG_TRIVIAL(error) << "发生错误";

    return 0;
}
```

---

## 自定义严重级别

```cpp
#include <boost/log/core.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>
#include <iostream>

namespace logging = boost::log;
namespace src = boost::log::sources;

enum severity_level {
    normal,
    notification,
    warning,
    error,
    critical
};

BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", severity_level)

void init_logging() {
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format = "[%Severity%] %Message%"
    );
    logging::add_common_attributes();
}

int main() {
    init_logging();

    src::severity_logger<severity_level> lg;

    BOOST_LOG_SEV(lg, normal) << "普通消息";
    BOOST_LOG_SEV(lg, notification) << "通知";
    BOOST_LOG_SEV(lg, warning) << "警告";
    BOOST_LOG_SEV(lg, error) << "错误";
    BOOST_LOG_SEV(lg, critical) << "严重错误";

    return 0;
}
```

---

## 格式化输出

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>
#include <boost/log/expressions.hpp>

namespace logging = boost::log;
namespace expr = boost::log::expressions;

void init_logging() {
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format = 
            expr::stream
                << "[" << expr::format_date_time<boost::posix_time::ptime>(
                    "TimeStamp", "%Y-%m-%d %H:%M:%S") << "]"
                << " [" << logging::trivial::severity << "]"
                << " [" << expr::attr<unsigned int>("LineID") << "]"
                << " " << expr::smessage
    );

    logging::add_common_attributes();
}

int main() {
    init_logging();

    BOOST_LOG_TRIVIAL(info) << "应用启动";
    BOOST_LOG_TRIVIAL(debug) << "调试信息";
    BOOST_LOG_TRIVIAL(warning) << "警告信息";

    return 0;
}
```

---

## 多个输出目标

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/utility/setup/file.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

namespace logging = boost::log;

void init_logging() {
    // 控制台输出
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format = "[%Severity%] %Message%"
    );

    // 文件输出
    logging::add_file_log(
        boost::log::keywords::file_name = "app.log",
        boost::log::keywords::format = "[%TimeStamp%] [%Severity%] %Message%"
    );

    logging::add_common_attributes();
}

int main() {
    init_logging();

    BOOST_LOG_TRIVIAL(info) << "同时输出到控制台和文件";
    BOOST_LOG_TRIVIAL(warning) << "警告消息";

    return 0;
}
```

---

## 过滤器

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

namespace logging = boost::log;

void init_logging() {
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format = "[%Severity%] %Message%"
    );

    // 只记录 warning 及以上级别
    logging::core::get()->set_filter(
        logging::trivial::severity >= logging::trivial::warning
    );

    logging::add_common_attributes();
}

int main() {
    init_logging();

    BOOST_LOG_TRIVIAL(trace) << "不会显示";
    BOOST_LOG_TRIVIAL(debug) << "不会显示";
    BOOST_LOG_TRIVIAL(info) << "不会显示";
    BOOST_LOG_TRIVIAL(warning) << "会显示";
    BOOST_LOG_TRIVIAL(error) << "会显示";

    return 0;
}
```

---

## 添加属性

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>
#include <boost/log/attributes.hpp>
#include <boost/log/expressions.hpp>

namespace logging = boost::log;
namespace src = boost::log::sources;
namespace expr = boost::log::expressions;
namespace attrs = boost::log::attributes;

BOOST_LOG_ATTRIBUTE_KEYWORD(line_id, "LineID", unsigned int)
BOOST_LOG_ATTRIBUTE_KEYWORD(timestamp, "TimeStamp", boost::posix_time::ptime)
BOOST_LOG_ATTRIBUTE_KEYWORD(scope, "Scope", std::string)

void init_logging() {
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format =
            expr::stream << "[" << line_id << "] "
                        << "[" << timestamp << "] "
                        << "[" << scope << "] "
                        << expr::smessage
    );

    logging::add_common_attributes();
}

int main() {
    init_logging();

    src::severity_logger<logging::trivial::severity_level> lg;
    lg.add_attribute("Scope", attrs::constant<std::string>("Main"));

    BOOST_LOG_TRIVIAL(info) << "带属性的日志";

    return 0;
}
```

---

## 异步日志

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/utility/setup/file.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>
#include <boost/log/sinks/async_frontend.hpp>
#include <boost/log/sinks/text_ostream_backend.hpp>
#include <boost/shared_ptr.hpp>
#include <fstream>

namespace logging = boost::log;
namespace sinks = boost::log::sinks;
namespace keywords = boost::log::keywords;

void init_async_logging() {
    typedef sinks::asynchronous_sink<
        sinks::text_ostream_backend
    > text_sink;

    boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

    boost::shared_ptr<std::ofstream> stream(
        new std::ofstream("async.log")
    );

    sink->locked_backend()->add_stream(stream);
    sink->set_formatter(
        logging::expressions::stream << "[" << logging::trivial::severity << "] "
                                    << logging::expressions::smessage
    );

    logging::core::get()->add_sink(sink);
    logging::add_common_attributes();
}

int main() {
    init_async_logging();

    for (int i = 0; i < 1000; ++i) {
        BOOST_LOG_TRIVIAL(info) << "异步日志消息 " << i;
    }

    return 0;
}
```

---

## 命名日志

```cpp
#include <boost/log/core.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>
#include <boost/log/attributes.hpp>
#include <string>

namespace logging = boost::log;
namespace src = boost::log::sources;
namespace attrs = boost::log::attributes;

BOOST_LOG_ATTRIBUTE_KEYWORD(channel, "Channel", std::string)

void init_logging() {
    logging::add_console_log(
        std::cout,
        boost::log::keywords::format = "[%Channel%] %Message%"
    );
    logging::add_common_attributes();
}

int main() {
    init_logging();

    src::logger network_logger;
    network_logger.add_attribute("Channel", attrs::constant<std::string>("Network"));

    src::logger database_logger;
    database_logger.add_attribute("Channel", attrs::constant<std::string>("Database"));

    BOOST_LOG(network_logger) << "网络连接建立";
    BOOST_LOG(database_logger) << "数据库查询执行";
    BOOST_LOG(network_logger) << "接收到数据";

    return 0;
}
```

---

## 参考资源

- [Boost.Log 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/log/doc/html/index.html)
- [日志教程](https://www.boost.org/doc/libs/1_90_0/libs/log/doc/html/log/tutorial.html)

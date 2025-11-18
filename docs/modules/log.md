# Boost.Log - 日志库

## 概述

Boost.Log 提供功能强大、高性能的日志记录框架。

**类型**: 需要编译链接的库

**链接库**: `-lboost_log -lboost_log_setup -lboost_thread -lboost_system -lpthread`

---

## 快速开始

```cpp
#include <boost/log/trivial.hpp>

int main() {
    BOOST_LOG_TRIVIAL(trace) << "A trace message";
    BOOST_LOG_TRIVIAL(debug) << "A debug message";
    BOOST_LOG_TRIVIAL(info) << "An info message";
    BOOST_LOG_TRIVIAL(warning) << "A warning message";
    BOOST_LOG_TRIVIAL(error) << "An error message";
    BOOST_LOG_TRIVIAL(fatal) << "A fatal message";

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_log -lboost_log_setup -lboost_thread -lboost_system -lpthread -DBOOST_LOG_DYN_LINK -o example
```

---

## 基本配置

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/utility/setup/file.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

namespace logging = boost::log;
namespace keywords = boost::log::keywords;

void init_logging() {
    // 添加控制台输出
    logging::add_console_log(
        std::cout,
        keywords::format = "[%TimeStamp%] [%Severity%] %Message%"
    );

    // 添加文件输出
    logging::add_file_log(
        keywords::file_name = "app_%N.log",
        keywords::rotation_size = 10 * 1024 * 1024,  // 10 MB
        keywords::time_based_rotation =
            logging::sinks::file::rotation_at_time_point(0, 0, 0),  // 每天轮换
        keywords::format = "[%TimeStamp%] [%Severity%] %Message%"
    );

    // 设置最小日志级别
    logging::core::get()->set_filter(
        logging::trivial::severity >= logging::trivial::info
    );

    // 添加通用属性
    logging::add_common_attributes();
}

int main() {
    init_logging();

    BOOST_LOG_TRIVIAL(debug) << "This won't appear";
    BOOST_LOG_TRIVIAL(info) << "Application started";
    BOOST_LOG_TRIVIAL(warning) << "This is a warning";
    BOOST_LOG_TRIVIAL(error) << "An error occurred";

    return 0;
}
```

---

## 自定义日志源

```cpp
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

namespace logging = boost::log;
namespace src = boost::log::sources;

enum severity_level {
    trace,
    debug,
    info,
    warning,
    error,
    fatal
};

int main() {
    logging::add_console_log(std::cout);
    logging::add_common_attributes();

    src::severity_logger<severity_level> lg;

    BOOST_LOG_SEV(lg, trace) << "A trace message";
    BOOST_LOG_SEV(lg, debug) << "A debug message";
    BOOST_LOG_SEV(lg, info) << "An info message";
    BOOST_LOG_SEV(lg, warning) << "A warning message";
    BOOST_LOG_SEV(lg, error) << "An error message";
    BOOST_LOG_SEV(lg, fatal) << "A fatal message";

    return 0;
}
```

---

## 结构化日志

```cpp
#include <boost/log/sources/logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/log/attributes/constant.hpp>
#include <boost/log/utility/setup/console.hpp>

namespace logging = boost::log;
namespace src = boost::log::sources;
namespace attrs = boost::log::attributes;

class AppLogger {
public:
    AppLogger(const std::string& module) {
        logger_.add_attribute("Module", attrs::constant<std::string>(module));
    }

    void log_info(const std::string& message) {
        BOOST_LOG(logger_) << "[INFO] " << message;
    }

    void log_error(const std::string& message) {
        BOOST_LOG(logger_) << "[ERROR] " << message;
    }

private:
    src::logger logger_;
};

int main() {
    logging::add_console_log(std::cout);

    AppLogger network_logger("Network");
    AppLogger database_logger("Database");

    network_logger.log_info("Connection established");
    database_logger.log_info("Query executed");
    network_logger.log_error("Connection lost");

    return 0;
}
```

---

## 过滤

```cpp
#include <boost/log/core.hpp>
#include <boost/log/trivial.hpp>
#include <boost/log/expressions.hpp>

namespace logging = boost::log;

int main() {
    // 只记录 warning 及以上级别
    logging::core::get()->set_filter(
        logging::trivial::severity >= logging::trivial::warning
    );

    BOOST_LOG_TRIVIAL(debug) << "Not logged";
    BOOST_LOG_TRIVIAL(info) << "Not logged";
    BOOST_LOG_TRIVIAL(warning) << "Logged!";
    BOOST_LOG_TRIVIAL(error) << "Logged!";

    return 0;
}
```

---

## 异步日志

```cpp
#include <boost/log/core.hpp>
#include <boost/log/sinks/async_frontend.hpp>
#include <boost/log/sinks/text_ostream_backend.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/log/sources/record_ostream.hpp>
#include <boost/shared_ptr.hpp>
#include <fstream>

namespace logging = boost::log;
namespace sinks = boost::log::sinks;
namespace src = boost::log::sources;

void init_async_logging() {
    typedef sinks::asynchronous_sink<
        sinks::text_ostream_backend
    > async_sink_t;

    auto backend = boost::make_shared<sinks::text_ostream_backend>();
    backend->add_stream(
        boost::shared_ptr<std::ostream>(new std::ofstream("async.log")));

    auto sink = boost::make_shared<async_sink_t>(backend);

    logging::core::get()->add_sink(sink);
}

int main() {
    init_async_logging();

    src::logger lg;

    for (int i = 0; i < 1000; ++i) {
        BOOST_LOG(lg) << "Message " << i;
    }

    return 0;
}
```

---

## 日志轮换

```cpp
#include <boost/log/utility/setup/file.hpp>
#include <boost/log/trivial.hpp>

namespace logging = boost::log;
namespace keywords = boost::log::keywords;

void init_rotating_log() {
    logging::add_file_log(
        keywords::file_name = "logs/app_%Y%m%d_%H%M%S_%N.log",
        keywords::rotation_size = 1 * 1024 * 1024,  // 1 MB
        keywords::max_size = 10 * 1024 * 1024,      // 总共 10 MB
        keywords::time_based_rotation =
            logging::sinks::file::rotation_at_time_point(12, 0, 0),  // 每天中午
        keywords::auto_flush = true
    );
}

int main() {
    init_rotating_log();

    for (int i = 0; i < 100000; ++i) {
        BOOST_LOG_TRIVIAL(info) << "Log entry " << i;
    }

    return 0;
}
```

---

## 最佳实践

1. **异步日志**: 高性能场景使用异步
2. **日志级别**: 生产环境使用 info 及以上
3. **日志轮换**: 避免日志文件过大
4. **结构化**: 使用属性添加上下文
5. **性能**: 避免在热路径记录 debug 日志

---

## 参考资源

- [Boost.Log 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/log/doc/html/index.html)

# Boost.System - 系统错误处理库

## 概述

Boost.System 提供操作系统错误代码的封装和处理机制，是许多 Boost 库的基础。

**类型**: 需要编译链接的库

**链接库**: `-lboost_system`

**注意**: C++11 引入了 `std::error_code`，但 Boost.System 提供了更多功能

---

## 快速开始

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>

namespace sys = boost::system;

int main() {
    // 创建错误代码
    sys::error_code ec;

    if (!ec) {
        std::cout << "没有错误" << std::endl;
    }

    // 创建一个错误
    ec = sys::errc::make_error_code(sys::errc::invalid_argument);

    std::cout << "错误代码: " << ec.value() << std::endl;
    std::cout << "错误消息: " << ec.message() << std::endl;
    std::cout << "错误类别: " << ec.category().name() << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_system -o example
```

---

## 系统错误码

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>
#include <fstream>

namespace sys = boost::system;

void read_file(const std::string& filename, sys::error_code& ec) {
    std::ifstream file(filename);

    if (!file) {
        ec = sys::errc::make_error_code(sys::errc::no_such_file_or_directory);
        return;
    }

    ec.clear();  // 成功
}

int main() {
    sys::error_code ec;

    read_file("nonexistent.txt", ec);

    if (ec) {
        std::cout << "读取文件失败: " << ec.message() << std::endl;
        std::cout << "错误值: " << ec.value() << std::endl;
    } else {
        std::cout << "读取成功" << std::endl;
    }

    return 0;
}
```

---

## 自定义错误类别

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>
#include <string>

namespace sys = boost::system;

// 定义自定义错误枚举
enum class MyError {
    success = 0,
    invalid_input = 1,
    operation_failed = 2,
    timeout = 3
};

// 自定义错误类别
class MyErrorCategory : public sys::error_category {
public:
    const char* name() const noexcept override {
        return "MyError";
    }

    std::string message(int ev) const override {
        switch (static_cast<MyError>(ev)) {
            case MyError::success:
                return "成功";
            case MyError::invalid_input:
                return "无效的输入";
            case MyError::operation_failed:
                return "操作失败";
            case MyError::timeout:
                return "超时";
            default:
                return "未知错误";
        }
    }
};

// 获取类别单例
const MyErrorCategory& my_error_category() {
    static MyErrorCategory instance;
    return instance;
}

// 创建错误代码的辅助函数
sys::error_code make_error_code(MyError e) {
    return sys::error_code(static_cast<int>(e), my_error_category());
}

// 让 Boost.System 识别自定义错误
namespace boost {
namespace system {
    template<>
    struct is_error_code_enum<MyError> : std::true_type {};
}
}

int main() {
    sys::error_code ec = MyError::invalid_input;

    std::cout << "错误: " << ec.message() << std::endl;
    std::cout << "类别: " << ec.category().name() << std::endl;
    std::cout << "值: " << ec.value() << std::endl;

    if (ec == MyError::invalid_input) {
        std::cout << "确实是无效输入错误" << std::endl;
    }

    return 0;
}
```

---

## 错误条件

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>

namespace sys = boost::system;

int main() {
    // 错误代码
    sys::error_code ec1 = sys::errc::make_error_code(sys::errc::permission_denied);
    sys::error_code ec2 = sys::errc::make_error_code(sys::errc::no_such_file_or_directory);

    // 错误条件
    sys::error_condition cond = sys::errc::make_error_condition(sys::errc::permission_denied);

    // 比较错误代码和错误条件
    if (ec1 == cond) {
        std::cout << "ec1 匹配权限被拒绝条件" << std::endl;
    }

    if (ec2 != cond) {
        std::cout << "ec2 不匹配权限被拒绝条件" << std::endl;
    }

    return 0;
}
```

---

## 异常和错误码

```cpp
#include <boost/system/error_code.hpp>
#include <boost/system/system_error.hpp>
#include <iostream>

namespace sys = boost::system;

void process_with_exception(bool should_fail) {
    if (should_fail) {
        throw sys::system_error(
            sys::errc::make_error_code(sys::errc::invalid_argument),
            "处理失败"
        );
    }
}

void process_with_error_code(bool should_fail, sys::error_code& ec) {
    if (should_fail) {
        ec = sys::errc::make_error_code(sys::errc::invalid_argument);
    } else {
        ec.clear();
    }
}

int main() {
    // 使用异常
    try {
        process_with_exception(true);
    } catch (const sys::system_error& e) {
        std::cout << "捕获异常: " << e.what() << std::endl;
        std::cout << "错误代码: " << e.code().value() << std::endl;
    }

    // 使用错误码
    sys::error_code ec;
    process_with_error_code(true, ec);

    if (ec) {
        std::cout << "处理失败: " << ec.message() << std::endl;
    }

    return 0;
}
```

---

## POSIX 错误码

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>
#include <cerrno>

namespace sys = boost::system;

int main() {
    // 从 POSIX errno 创建错误码
    errno = EACCES;
    sys::error_code ec(errno, sys::system_category());

    std::cout << "POSIX 错误: " << ec.message() << std::endl;

    // 使用 errc 枚举
    sys::error_code ec2 = sys::errc::make_error_code(sys::errc::permission_denied);
    std::cout << "errc 错误: " << ec2.message() << std::endl;

    // 比较
    if (ec == ec2) {
        std::cout << "两个错误码相同" << std::endl;
    }

    return 0;
}
```

---

## 错误处理策略

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>
#include <fstream>

namespace sys = boost::system;

class FileReader {
public:
    // 策略1: 返回错误码
    bool open(const std::string& filename, sys::error_code& ec) {
        file_.open(filename);
        if (!file_) {
            ec = sys::errc::make_error_code(sys::errc::no_such_file_or_directory);
            return false;
        }
        ec.clear();
        return true;
    }

    // 策略2: 抛出异常
    void open_throw(const std::string& filename) {
        file_.open(filename);
        if (!file_) {
            throw sys::system_error(
                sys::errc::make_error_code(sys::errc::no_such_file_or_directory),
                "无法打开文件: " + filename
            );
        }
    }

private:
    std::ifstream file_;
};

int main() {
    FileReader reader;

    // 使用错误码（不抛异常）
    sys::error_code ec;
    if (!reader.open("test.txt", ec)) {
        std::cout << "打开失败: " << ec.message() << std::endl;
    }

    // 使用异常
    try {
        reader.open_throw("test.txt");
    } catch (const sys::system_error& e) {
        std::cout << "异常: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 错误码转换

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>
#include <system_error>

namespace sys = boost::system;

int main() {
    // Boost.System 错误码
    sys::error_code boost_ec = sys::errc::make_error_code(sys::errc::invalid_argument);

    std::cout << "Boost 错误: " << boost_ec.message() << std::endl;

    // 转换为 std::error_code (C++11)
    std::error_code std_ec(boost_ec.value(), std::generic_category());

    std::cout << "std 错误: " << std_ec.message() << std::endl;

    return 0;
}
```

---

## 成功检查辅助函数

```cpp
#include <boost/system/error_code.hpp>
#include <iostream>

namespace sys = boost::system;

// 成功返回的函数
sys::error_code operation_success() {
    return sys::error_code();  // 默认构造表示成功
}

// 失败返回的函数
sys::error_code operation_failure() {
    return sys::errc::make_error_code(sys::errc::io_error);
}

int main() {
    sys::error_code ec1 = operation_success();
    sys::error_code ec2 = operation_failure();

    // 检查成功
    if (!ec1) {
        std::cout << "操作1成功" << std::endl;
    }

    // 检查失败
    if (ec2) {
        std::cout << "操作2失败: " << ec2.message() << std::endl;
    }

    // 显式检查
    if (ec1 == sys::errc::success) {
        std::cout << "操作1确实成功" << std::endl;
    }

    return 0;
}
```

---

## 与其他 Boost 库集成

```cpp
#include <boost/system/error_code.hpp>
#include <boost/filesystem.hpp>
#include <iostream>

namespace sys = boost::system;
namespace fs = boost::filesystem;

int main() {
    sys::error_code ec;

    // Boost.Filesystem 使用 Boost.System
    fs::path p = fs::current_path(ec);

    if (ec) {
        std::cout << "获取当前路径失败: " << ec.message() << std::endl;
    } else {
        std::cout << "当前路径: " << p << std::endl;
    }

    // 尝试创建目录
    fs::create_directory("/invalid/path", ec);

    if (ec) {
        std::cout << "创建目录失败: " << ec.message() << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.System 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/system/doc/html/index.html)
- [错误处理设计](https://www.boost.org/doc/libs/1_90_0/libs/system/doc/html/system/design.html)
- [C++11 std::error_code](https://en.cppreference.com/w/cpp/error/error_code)

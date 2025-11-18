# Boost C++ 库 1.90.0.beta1 完整使用指南

## 目录

1. [简介](#简介)
2. [快速开始](#快速开始)
3. [编译安装](#编译安装)
4. [使用方法](#使用方法)
5. [模块索引](#模块索引)
6. [常见问题](#常见问题)

---

## 简介

Boost 是一个高质量、可移植、开源的 C++ 库集合，包含 159 个独立但相互协作的库模块。Boost 库涵盖了从智能指针、容器、算法到网络编程、并发处理、数学计算等各个领域。

### 版本信息

- **版本**: 1.90.0.beta1
- **许可证**: Boost Software License 1.0（非常宽松的开源许可证）
- **C++ 标准**: 支持 C++11 及更高版本
- **平台支持**: Linux, Windows, macOS, 以及其他 UNIX 系统

### 核心特性

- **高质量**: 经过严格的代码审查和测试
- **可移植性**: 支持主流编译器和平台
- **标准化**: 许多 Boost 库已被纳入 C++ 标准库
- **模块化**: 可以选择性地使用所需的库

---

## 快速开始

### 系统要求

**编译器支持**:
- GCC 5.0 或更高版本
- Clang 3.4 或更高版本
- MSVC 14.0 (Visual Studio 2015) 或更高版本
- Intel C++ 17.0 或更高版本

**构建工具**:
- CMake 3.8 或更高版本（推荐 3.15+）
- 或 Boost.Build (b2/bjam)

**其他依赖**:
- Python 2.7 或 3.x（用于构建某些库）
- Git（用于获取子模块）

### 10 分钟上手

```bash
# 1. 克隆仓库
git clone --recursive https://github.com/boostorg/boost.git
cd boost

# 2. 检出特定版本
git checkout boost-1.90.0.beta1
git submodule update --init --recursive

# 3. 快速构建（使用 b2）
./bootstrap.sh --prefix=/usr/local
./b2 --with-system --with-filesystem stage

# 4. 在你的项目中使用
g++ -I./boost main.cpp -L./stage/lib -lboost_system -lboost_filesystem
```

---

## 编译安装

### 方法一：使用 CMake（推荐）

#### 基本安装

```bash
# 1. 初始化所有子模块
git submodule update --init --recursive

# 2. 创建构建目录
mkdir build && cd build

# 3. 配置 CMake
cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DCMAKE_CXX_STANDARD=17

# 4. 编译（使用所有 CPU 核心）
cmake --build . --parallel $(nproc)

# 5. 安装（可能需要 sudo）
sudo cmake --install .
```

#### 自定义安装选项

```bash
# 只编译特定的库
cmake .. \
    -DCMAKE_INSTALL_PREFIX=$HOME/boost \
    -DBOOST_INCLUDE_LIBRARIES="system;filesystem;thread;asio"

# 启用静态库
cmake .. \
    -DBUILD_SHARED_LIBS=OFF

# 设置 C++ 标准
cmake .. \
    -DCMAKE_CXX_STANDARD=20
```

### 方法二：使用 Boost.Build (b2)

#### Linux/macOS 安装

```bash
# 1. 引导构建系统
./bootstrap.sh --prefix=/usr/local --with-toolset=gcc

# 2. 编译所有库
./b2 stage \
    variant=release \
    link=shared,static \
    threading=multi \
    -j$(nproc)

# 3. 安装到系统
sudo ./b2 install \
    variant=release \
    link=shared,static \
    threading=multi
```

#### Windows 安装

```cmd
REM 1. 引导构建系统
bootstrap.bat

REM 2. 编译所有库
b2 stage ^
    variant=release ^
    link=shared,static ^
    threading=multi ^
    address-model=64

REM 3. 安装
b2 install ^
    variant=release ^
    --prefix=C:\boost
```

#### 选择性编译特定库

```bash
# 只编译需要的库（加快编译速度）
./b2 stage \
    --with-system \
    --with-filesystem \
    --with-thread \
    --with-date_time \
    --with-regex \
    --with-program_options \
    -j$(nproc)
```

#### b2 常用选项说明

| 选项 | 说明 | 可选值 |
|------|------|--------|
| `variant` | 编译类型 | `debug`, `release` |
| `link` | 链接方式 | `shared`, `static` |
| `threading` | 线程支持 | `single`, `multi` |
| `address-model` | 目标架构 | `32`, `64` |
| `toolset` | 编译器 | `gcc`, `clang`, `msvc` |
| `--prefix` | 安装路径 | 任意路径 |
| `-j` | 并行编译数 | 数字（建议 CPU 核心数） |

### 方法三：仅头文件库使用

很多 Boost 库是纯头文件库，无需编译：

```bash
# 1. 克隆仓库
git clone --recursive https://github.com/boostorg/boost.git

# 2. 直接使用头文件
g++ -I/path/to/boost main.cpp
```

**仅头文件的库**包括：
- Algorithm
- Any
- Array
- Bind
- Concept Check
- Container Hash
- Core
- Foreach
- Function
- Iterator
- Lambda
- MPL
- Optional
- Smart Ptr
- Static Assert
- Type Traits
- Utility
等等...

---

## 使用方法

### 在项目中使用 Boost

#### 使用 CMake

创建 `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.15)
project(MyBoostProject)

set(CMAKE_CXX_STANDARD 17)

# 查找 Boost 库
find_package(Boost 1.90 REQUIRED
    COMPONENTS
        system
        filesystem
        thread
        program_options
)

# 添加可执行文件
add_executable(myapp main.cpp)

# 链接 Boost 库
target_link_libraries(myapp
    Boost::system
    Boost::filesystem
    Boost::thread
    Boost::program_options
)

# 包含 Boost 头文件
target_include_directories(myapp PRIVATE ${Boost_INCLUDE_DIRS})
```

#### 使用 Makefile

```makefile
CXX = g++
CXXFLAGS = -std=c++17 -I/usr/local/include
LDFLAGS = -L/usr/local/lib
LIBS = -lboost_system -lboost_filesystem -lboost_thread -lpthread

myapp: main.cpp
	$(CXX) $(CXXFLAGS) main.cpp -o myapp $(LDFLAGS) $(LIBS)

clean:
	rm -f myapp
```

#### 使用 pkg-config

```bash
# 安装后，可以使用 pkg-config
g++ main.cpp -o myapp \
    $(pkg-config --cflags --libs boost_system boost_filesystem)
```

### 简单示例程序

创建 `main.cpp`:

```cpp
#include <iostream>
#include <boost/version.hpp>
#include <boost/config.hpp>
#include <boost/filesystem.hpp>
#include <boost/smart_ptr.hpp>

namespace fs = boost::filesystem;

int main() {
    // 显示 Boost 版本
    std::cout << "使用 Boost 版本: "
              << BOOST_VERSION / 100000 << "."
              << BOOST_VERSION / 100 % 1000 << "."
              << BOOST_VERSION % 100 << std::endl;

    // 使用智能指针
    boost::shared_ptr<int> ptr(new int(42));
    std::cout << "智能指针值: " << *ptr << std::endl;

    // 使用文件系统库
    fs::path current_path = fs::current_path();
    std::cout << "当前目录: " << current_path << std::endl;

    return 0;
}
```

编译运行：

```bash
g++ -std=c++17 -I/usr/local/include main.cpp \
    -L/usr/local/lib -lboost_filesystem -lboost_system -o myapp
./myapp
```

---

## 模块索引

Boost 1.90.0.beta1 包含 **159 个库模块**，按类别分类如下：

### 核心工具库

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Smart Ptr](modules/smart_ptr.md) | 智能指针（shared_ptr, unique_ptr 等） | 查看详情 |
| [Core](modules/core.md) | 核心工具和宏 | 查看详情 |
| [Config](modules/config.md) | 编译器配置和平台检测 | 查看详情 |
| [Assert](modules/assert.md) | 增强的断言宏 | 查看详情 |
| [Static Assert](modules/static_assert.md) | 编译期断言 | 查看详情 |
| [Type Traits](modules/type_traits.md) | 类型特征和元编程 | 查看详情 |
| [Utility](modules/utility.md) | 通用工具函数 | 查看详情 |
| [Exception](modules/exception.md) | 异常处理增强 | 查看详情 |

### 容器库

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Container](modules/container.md) | STL 兼容的高级容器 | 查看详情 |
| [Array](modules/array.md) | 固定大小数组 | 查看详情 |
| [Unordered](modules/unordered.md) | 无序关联容器 | 查看详情 |
| [Multi-Index](modules/multi_index.md) | 多索引容器 | 查看详情 |
| [Circular Buffer](modules/circular_buffer.md) | 循环缓冲区 | 查看详情 |
| [Heap](modules/heap.md) | 堆数据结构 | 查看详情 |
| [Intrusive](modules/intrusive.md) | 侵入式容器 | 查看详情 |
| [Ptr Container](modules/ptr_container.md) | 指针容器 | 查看详情 |

### 字符串和文本处理

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Regex](modules/regex.md) | 正则表达式 | 查看详情 |
| [Spirit](modules/spirit.md) | 解析器框架 | 查看详情 |
| [Tokenizer](modules/tokenizer.md) | 字符串分词 | 查看详情 |
| [Lexical Cast](modules/lexical_cast.md) | 类型转换 | 查看详情 |
| [Format](modules/format.md) | 字符串格式化 | 查看详情 |
| [String Algo](modules/algorithm.md) | 字符串算法 | 查看详情 |
| [JSON](modules/json.md) | JSON 解析和生成 | 查看详情 |
| [Locale](modules/locale.md) | 本地化支持 | 查看详情 |

### 并发和异步

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Asio](modules/asio.md) | 异步 I/O 和网络编程 | 查看详情 |
| [Thread](modules/thread.md) | 线程管理 | 查看详情 |
| [Atomic](modules/atomic.md) | 原子操作 | 查看详情 |
| [Lockfree](modules/lockfree.md) | 无锁数据结构 | 查看详情 |
| [Fiber](modules/fiber.md) | 用户态线程（纤程） | 查看详情 |
| [Coroutine2](modules/coroutine2.md) | 协程支持 | 查看详情 |
| [Context](modules/context.md) | 上下文切换 | 查看详情 |

### 网络和 I/O

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Asio](modules/asio.md) | 异步网络和 I/O | 查看详情 |
| [Beast](modules/beast.md) | HTTP/WebSocket 库 | 查看详情 |
| [Filesystem](modules/filesystem.md) | 文件系统操作 | 查看详情 |
| [Iostreams](modules/iostreams.md) | I/O 流扩展 | 查看详情 |
| [Process](modules/process.md) | 进程管理 | 查看详情 |
| [URL](modules/url.md) | URL 解析 | 查看详情 |

### 数学和数值计算

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Math](modules/math.md) | 数学函数和统计 | 查看详情 |
| [Multiprecision](modules/multiprecision.md) | 多精度数值 | 查看详情 |
| [Random](modules/random.md) | 随机数生成 | 查看详情 |
| [Numeric](modules/numeric.md) | 数值算法 | 查看详情 |
| [Rational](modules/rational.md) | 有理数运算 | 查看详情 |
| [Geometry](modules/geometry.md) | 几何计算 | 查看详情 |

### 日期和时间

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Date Time](modules/date_time.md) | 日期时间处理 | 查看详情 |
| [Chrono](modules/chrono.md) | 时间点和时长 | 查看详情 |
| [Timer](modules/timer.md) | 计时器 | 查看详情 |

### 函数式编程

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Function](modules/function.md) | 函数对象封装 | 查看详情 |
| [Bind](modules/bind.md) | 函数绑定 | 查看详情 |
| [Lambda](modules/lambda.md) | Lambda 表达式 | 查看详情 |
| [Phoenix](modules/phoenix.md) | 函数式编程库 | 查看详情 |
| [Signals2](modules/signals2.md) | 信号/槽机制 | 查看详情 |

### 元编程

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [MPL](modules/mpl.md) | 模板元编程库 | 查看详情 |
| [MP11](modules/mp11.md) | C++11 元编程 | 查看详情 |
| [Hana](modules/hana.md) | 异构编译时编程 | 查看详情 |
| [Fusion](modules/fusion.md) | 编译时和运行时序列 | 查看详情 |
| [Preprocessor](modules/preprocessor.md) | 预处理器元编程 | 查看详情 |

### 错误处理

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [System](modules/system.md) | 错误码和系统错误 | 查看详情 |
| [Outcome](modules/outcome.md) | 结果类型（类似 Rust） | 查看详情 |
| [LEAF](modules/leaf.md) | 轻量级错误处理 | 查看详情 |

### 其他专用库

| 模块 | 说明 | 文档链接 |
|------|------|----------|
| [Program Options](modules/program_options.md) | 命令行参数解析 | 查看详情 |
| [Serialization](modules/serialization.md) | 对象序列化 | 查看详情 |
| [Test](modules/test.md) | 单元测试框架 | 查看详情 |
| [Log](modules/log.md) | 日志库 | 查看详情 |
| [UUID](modules/uuid.md) | 唯一标识符生成 | 查看详情 |
| [CRC](modules/crc.md) | CRC 校验 | 查看详情 |
| [Graph](modules/graph.md) | 图论算法 | 查看详情 |

**完整模块列表**: 查看 [所有 159 个模块索引](modules/INDEX.md)

---

## 常见问题

### 编译问题

**Q: 编译时提示找不到某些库？**

A: 确保已经初始化所有子模块：
```bash
git submodule update --init --recursive
```

**Q: 编译速度太慢？**

A: 使用并行编译和只编译需要的库：
```bash
./b2 --with-system --with-filesystem -j8
```

**Q: CMake 找不到 Boost？**

A: 设置 `BOOST_ROOT` 环境变量：
```bash
export BOOST_ROOT=/path/to/boost
cmake ..
```

### 链接问题

**Q: 链接时提示 undefined reference？**

A: 确保链接了正确的库文件：
```bash
g++ main.cpp -lboost_system -lboost_filesystem
```

**Q: 如何判断一个库是否需要编译链接？**

A: 查看头文件中是否有 `#if defined(BOOST_ALL_NO_LIB)` 等宏。通常需要链接的库有：
- system
- filesystem
- thread
- regex
- program_options
- date_time
- iostreams
- serialization
等

### 使用问题

**Q: 如何查看 Boost 版本？**

A: 在代码中：
```cpp
#include <boost/version.hpp>
std::cout << BOOST_LIB_VERSION << std::endl;
```

**Q: 可以在商业项目中使用吗？**

A: 可以！Boost 使用非常宽松的 Boost Software License 1.0，允许商业使用。

**Q: Boost 和 C++ 标准库的关系？**

A: 许多 Boost 库已被纳入 C++ 标准，如：
- `boost::shared_ptr` → `std::shared_ptr`
- `boost::unordered_map` → `std::unordered_map`
- `boost::regex` → `std::regex`
- `boost::filesystem` → `std::filesystem` (C++17)
- `boost::optional` → `std::optional` (C++17)

---

## 获取帮助

- **官方网站**: https://www.boost.org/
- **官方文档**: https://www.boost.org/doc/libs/1_90_0/
- **GitHub 仓库**: https://github.com/boostorg/boost
- **邮件列表**: https://lists.boost.org/
- **Stack Overflow**: 使用 `boost` 标签

---

## 贡献

欢迎为 Boost 贡献代码！请访问 [贡献指南](https://github.com/boostorg/boost/wiki/Getting-Started)

---

**版权声明**: 本文档基于 Boost 1.90.0.beta1 版本编写，遵循 Boost Software License 1.0。

# Boost.Filesystem - 文件系统库

## 概述

Boost.Filesystem 提供了跨平台的文件系统操作功能，可以方便地进行文件、目录的创建、删除、遍历等操作。

**类型**: 需要编译链接的库

**链接库**: `-lboost_filesystem -lboost_system`

**注意**: C++17 已将此库纳入标准库 (`std::filesystem`)

---

## 快速开始

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    // 获取当前路径
    fs::path current = fs::current_path();
    std::cout << "当前路径: " << current << std::endl;

    // 检查文件是否存在
    if (fs::exists("test.txt")) {
        std::cout << "文件存在" << std::endl;
    }

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_filesystem -lboost_system -o example
```

---

## 路径操作

### 创建和操作路径

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    // 1. 创建路径
    fs::path p1("/home/user/document.txt");
    fs::path p2("relative/path/file.txt");

    // 2. 路径拼接
    fs::path dir = "/home/user";
    fs::path file = "data.txt";
    fs::path full = dir / file; // /home/user/data.txt

    std::cout << "完整路径: " << full << std::endl;

    // 3. 路径分解
    std::cout << "根路径: " << p1.root_path() << std::endl;          // /
    std::cout << "根名称: " << p1.root_name() << std::endl;          // (空)
    std::cout << "根目录: " << p1.root_directory() << std::endl;     // /
    std::cout << "相对路径: " << p1.relative_path() << std::endl;    // home/user/document.txt
    std::cout << "父路径: " << p1.parent_path() << std::endl;        // /home/user
    std::cout << "文件名: " << p1.filename() << std::endl;           // document.txt
    std::cout << "主干名: " << p1.stem() << std::endl;               // document
    std::cout << "扩展名: " << p1.extension() << std::endl;          // .txt

    // 4. 修改路径
    fs::path p3 = "/home/user/file.txt";
    p3.replace_extension(".md");
    std::cout << "新路径: " << p3 << std::endl; // /home/user/file.md

    // 5. 规范化路径
    fs::path p4 = "/home/user/../user/./file.txt";
    std::cout << "规范化: " << fs::canonical(p4) << std::endl;

    return 0;
}
```

### 路径转换

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <string>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "/home/user/文档/文件.txt";

    // 转换为字符串
    std::string str = p.string();
    std::cout << "string: " << str << std::endl;

    // 转换为宽字符串
    std::wstring wstr = p.wstring();

    // 通用格式（使用 /）
    std::string generic = p.generic_string();
    std::cout << "generic: " << generic << std::endl;

    return 0;
}
```

---

## 文件和目录查询

### 检查文件状态

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

void check_file_status(const fs::path& p) {
    std::cout << "\n检查: " << p << std::endl;

    // 1. 基本检查
    std::cout << "存在: " << fs::exists(p) << std::endl;

    if (!fs::exists(p)) {
        return;
    }

    // 2. 类型检查
    std::cout << "是常规文件: " << fs::is_regular_file(p) << std::endl;
    std::cout << "是目录: " << fs::is_directory(p) << std::endl;
    std::cout << "是符号链接: " << fs::is_symlink(p) << std::endl;
    std::cout << "是其他类型: " << fs::is_other(p) << std::endl;

    // 3. 文件大小
    if (fs::is_regular_file(p)) {
        std::cout << "文件大小: " << fs::file_size(p) << " 字节" << std::endl;
    }

    // 4. 最后修改时间
    std::time_t t = fs::last_write_time(p);
    std::cout << "最后修改: " << std::ctime(&t);

    // 5. 权限检查（Unix）
    fs::perms permissions = fs::status(p).permissions();
    std::cout << "可读: " << ((permissions & fs::owner_read) != fs::no_perms) << std::endl;
    std::cout << "可写: " << ((permissions & fs::owner_write) != fs::no_perms) << std::endl;
    std::cout << "可执行: " << ((permissions & fs::owner_exe) != fs::no_perms) << std::endl;
}

int main() {
    check_file_status("/etc/passwd");
    check_file_status("/tmp");
    check_file_status("/usr/bin/ls");

    return 0;
}
```

### 获取磁盘空间信息

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <iomanip>

namespace fs = boost::filesystem;

int main() {
    fs::space_info si = fs::space(".");

    std::cout << "磁盘空间信息:\n";
    std::cout << "总容量: " << si.capacity / (1024*1024*1024) << " GB\n";
    std::cout << "可用空间: " << si.available / (1024*1024*1024) << " GB\n";
    std::cout << "剩余空间: " << si.free / (1024*1024*1024) << " GB\n";

    double usage = 100.0 * (si.capacity - si.free) / si.capacity;
    std::cout << "使用率: " << std::fixed << std::setprecision(2)
              << usage << "%\n";

    return 0;
}
```

---

## 文件和目录操作

### 创建目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    // 1. 创建单个目录
    fs::path dir1 = "test_dir";
    if (fs::create_directory(dir1)) {
        std::cout << "创建目录: " << dir1 << std::endl;
    }

    // 2. 创建多级目录
    fs::path dir2 = "parent/child/grandchild";
    if (fs::create_directories(dir2)) {
        std::cout << "创建目录树: " << dir2 << std::endl;
    }

    // 3. 检查是否成功
    if (fs::exists(dir2) && fs::is_directory(dir2)) {
        std::cout << "目录创建成功" << std::endl;
    }

    return 0;
}
```

### 复制文件和目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>

namespace fs = boost::filesystem;

int main() {
    // 1. 创建测试文件
    std::ofstream("source.txt") << "Hello, Boost.Filesystem!";

    // 2. 复制文件
    fs::copy_file("source.txt", "dest.txt",
                  fs::copy_option::overwrite_if_exists);

    std::cout << "文件已复制" << std::endl;

    // 3. 复制目录（递归）
    fs::path src_dir = "source_dir";
    fs::path dst_dir = "dest_dir";

    fs::create_directory(src_dir);
    std::ofstream(src_dir / "file.txt") << "test";

    if (!fs::exists(dst_dir)) {
        fs::create_directory(dst_dir);
    }

    // 递归复制
    for (fs::recursive_directory_iterator it(src_dir), end; it != end; ++it) {
        fs::path rel = fs::relative(it->path(), src_dir);
        fs::path dst = dst_dir / rel;

        if (fs::is_directory(it->path())) {
            fs::create_directory(dst);
        } else {
            fs::copy_file(it->path(), dst, fs::copy_option::overwrite_if_exists);
        }
    }

    std::cout << "目录已复制" << std::endl;

    return 0;
}
```

### 移动和重命名

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>

namespace fs = boost::filesystem;

int main() {
    // 创建测试文件
    std::ofstream("old_name.txt") << "test content";

    // 1. 重命名文件
    fs::rename("old_name.txt", "new_name.txt");
    std::cout << "文件已重命名" << std::endl;

    // 2. 移动文件到其他目录
    fs::create_directory("target_dir");
    fs::rename("new_name.txt", "target_dir/moved_file.txt");
    std::cout << "文件已移动" << std::endl;

    return 0;
}
```

### 删除文件和目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>

namespace fs = boost::filesystem;

int main() {
    // 1. 删除文件
    std::ofstream("temp.txt") << "temporary";

    if (fs::remove("temp.txt")) {
        std::cout << "文件已删除" << std::endl;
    }

    // 2. 删除空目录
    fs::create_directory("empty_dir");
    if (fs::remove("empty_dir")) {
        std::cout << "空目录已删除" << std::endl;
    }

    // 3. 递归删除目录（包含内容）
    fs::path dir = "dir_with_files";
    fs::create_directories(dir / "subdir");
    std::ofstream(dir / "file.txt") << "test";
    std::ofstream(dir / "subdir" / "file2.txt") << "test2";

    std::uintmax_t n = fs::remove_all(dir);
    std::cout << "删除了 " << n << " 个文件/目录" << std::endl;

    return 0;
}
```

---

## 目录遍历

### 基本目录遍历

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path dir = ".";

    std::cout << "遍历目录: " << fs::absolute(dir) << "\n\n";

    // 遍历当前目录
    for (fs::directory_iterator it(dir), end; it != end; ++it) {
        std::cout << it->path().filename() << std::endl;

        // 显示详细信息
        if (fs::is_regular_file(it->path())) {
            std::cout << "  [文件] 大小: " << fs::file_size(it->path()) << " 字节\n";
        } else if (fs::is_directory(it->path())) {
            std::cout << "  [目录]\n";
        } else if (fs::is_symlink(it->path())) {
            std::cout << "  [符号链接]\n";
        }
    }

    return 0;
}
```

### 递归目录遍历

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <iomanip>

namespace fs = boost::filesystem;

void list_directory_tree(const fs::path& dir, int indent = 0) {
    for (fs::directory_iterator it(dir), end; it != end; ++it) {
        // 打印缩进
        std::cout << std::string(indent * 2, ' ');

        // 打印文件名
        std::cout << it->path().filename();

        if (fs::is_directory(it->path())) {
            std::cout << "/\n";
            // 递归遍历子目录
            list_directory_tree(it->path(), indent + 1);
        } else {
            std::cout << " (" << fs::file_size(it->path()) << " bytes)\n";
        }
    }
}

int main() {
    fs::path dir = ".";
    std::cout << "目录树:\n";
    std::cout << dir.filename() << "/\n";
    list_directory_tree(dir, 1);

    return 0;
}
```

### 使用 recursive_directory_iterator

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path root = ".";

    std::cout << "递归遍历所有文件:\n";

    for (fs::recursive_directory_iterator it(root), end; it != end; ++it) {
        // 获取相对路径
        fs::path rel = fs::relative(it->path(), root);

        // 显示层级
        int level = std::distance(rel.begin(), rel.end()) - 1;
        std::cout << std::string(level * 2, ' ') << rel.filename();

        if (fs::is_directory(it->path())) {
            std::cout << "/\n";
        } else {
            std::cout << " - " << fs::file_size(it->path()) << " bytes\n";
        }
    }

    return 0;
}
```

### 过滤文件

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <vector>

namespace fs = boost::filesystem;

// 查找所有特定扩展名的文件
std::vector<fs::path> find_files_by_extension(
    const fs::path& dir,
    const std::string& ext
) {
    std::vector<fs::path> result;

    for (fs::recursive_directory_iterator it(dir), end; it != end; ++it) {
        if (fs::is_regular_file(it->path()) &&
            it->path().extension() == ext) {
            result.push_back(it->path());
        }
    }

    return result;
}

int main() {
    // 查找所有 .cpp 文件
    auto cpp_files = find_files_by_extension(".", ".cpp");

    std::cout << "找到 " << cpp_files.size() << " 个 .cpp 文件:\n";
    for (const auto& f : cpp_files) {
        std::cout << "  " << f << std::endl;
    }

    return 0;
}
```

---

## 实用示例

### 计算目录大小

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

std::uintmax_t calculate_directory_size(const fs::path& dir) {
    std::uintmax_t size = 0;

    for (fs::recursive_directory_iterator it(dir), end; it != end; ++it) {
        if (fs::is_regular_file(it->path())) {
            size += fs::file_size(it->path());
        }
    }

    return size;
}

int main() {
    fs::path dir = ".";

    std::uintmax_t size = calculate_directory_size(dir);

    std::cout << "目录大小: ";
    if (size > 1024 * 1024 * 1024) {
        std::cout << (size / (1024.0 * 1024 * 1024)) << " GB\n";
    } else if (size > 1024 * 1024) {
        std::cout << (size / (1024.0 * 1024)) << " MB\n";
    } else if (size > 1024) {
        std::cout << (size / 1024.0) << " KB\n";
    } else {
        std::cout << size << " bytes\n";
    }

    return 0;
}
```

### 清理临时文件

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <chrono>

namespace fs = boost::filesystem;

// 删除超过指定天数的文件
void cleanup_old_files(const fs::path& dir, int days) {
    auto now = std::chrono::system_clock::now();
    auto threshold = now - std::chrono::hours(24 * days);

    for (fs::recursive_directory_iterator it(dir), end; it != end; ++it) {
        if (fs::is_regular_file(it->path())) {
            auto ftime = fs::last_write_time(it->path());
            auto sctp = std::chrono::system_clock::from_time_t(ftime);

            if (sctp < threshold) {
                std::cout << "删除旧文件: " << it->path() << std::endl;
                fs::remove(it->path());
            }
        }
    }
}

int main() {
    cleanup_old_files("/tmp", 7); // 删除 7 天前的文件
    return 0;
}
```

### 备份目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <sstream>
#include <ctime>

namespace fs = boost::filesystem;

fs::path create_backup(const fs::path& source) {
    // 生成时间戳
    std::time_t t = std::time(nullptr);
    std::tm* tm = std::localtime(&t);

    std::ostringstream oss;
    oss << source.filename().string() << "_backup_"
        << (tm->tm_year + 1900) << "-"
        << (tm->tm_mon + 1) << "-"
        << tm->tm_mday << "_"
        << tm->tm_hour << "-"
        << tm->tm_min << "-"
        << tm->tm_sec;

    fs::path backup = source.parent_path() / oss.str();

    // 递归复制
    fs::copy_directory(source, backup);

    for (fs::recursive_directory_iterator it(source), end; it != end; ++it) {
        fs::path rel = fs::relative(it->path(), source);
        fs::path dst = backup / rel;

        if (fs::is_directory(it->path())) {
            fs::create_directory(dst);
        } else {
            fs::copy_file(it->path(), dst);
        }
    }

    return backup;
}

int main() {
    fs::path source = "important_data";

    try {
        fs::path backup = create_backup(source);
        std::cout << "备份创建成功: " << backup << std::endl;
    } catch (const fs::filesystem_error& e) {
        std::cerr << "备份失败: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 错误处理

### 使用异常

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    try {
        fs::path p = "nonexistent_file.txt";

        // 这会抛出异常
        std::uintmax_t size = fs::file_size(p);

    } catch (const fs::filesystem_error& e) {
        std::cerr << "文件系统错误: " << e.what() << std::endl;
        std::cerr << "路径1: " << e.path1() << std::endl;
        std::cerr << "路径2: " << e.path2() << std::endl;
        std::cerr << "错误码: " << e.code() << std::endl;
    }

    return 0;
}
```

### 使用错误码

```cpp
#include <boost/filesystem.hpp>
#include <boost/system/error_code.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "nonexistent_file.txt";
    boost::system::error_code ec;

    // 使用错误码版本（不抛异常）
    std::uintmax_t size = fs::file_size(p, ec);

    if (ec) {
        std::cerr << "错误: " << ec.message() << std::endl;
    } else {
        std::cout << "文件大小: " << size << std::endl;
    }

    return 0;
}
```

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_filesystem -lboost_system -o example

# 使用 CMake
find_package(Boost REQUIRED COMPONENTS filesystem system)
target_link_libraries(myapp Boost::filesystem Boost::system)

# 静态链接
g++ -std=c++11 example.cpp \
    /usr/local/lib/libboost_filesystem.a \
    /usr/local/lib/libboost_system.a \
    -o example
```

---

## 最佳实践

1. **使用异常处理**: 文件系统操作容易出错，要做好异常处理
2. **路径拼接**: 使用 `/` 操作符而不是字符串拼接
3. **检查存在性**: 操作前检查文件/目录是否存在
4. **权限处理**: 注意文件权限，特别是在多用户系统
5. **C++17**: 如果可能，使用 `std::filesystem` 替代

---

## 参考资源

- [Boost.Filesystem 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/filesystem/doc/index.htm)
- [Boost.Filesystem 教程](https://www.boost.org/doc/libs/1_90_0/libs/filesystem/doc/tutorial.html)

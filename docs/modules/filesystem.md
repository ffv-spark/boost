# Boost.Filesystem - 文件系统库

## 概述

Boost.Filesystem 提供可移植的文件系统操作功能，是 C++17 std::filesystem 的前身。

**类型**: 需要编译的库

**注意**: C++17 引入了 std::filesystem，优先使用标准库版本

---

## 快速开始

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "/home/user/documents";

    std::cout << "路径: " << p << std::endl;
    std::cout << "文件名: " << p.filename() << std::endl;
    std::cout << "父路径: " << p.parent_path() << std::endl;

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_filesystem -lboost_system`

---

## 路径操作

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p1 = "/home/user";
    fs::path p2 = "documents";
    fs::path p3 = "file.txt";

    // 路径连接
    fs::path full = p1 / p2 / p3;
    std::cout << "完整路径: " << full << std::endl;

    // 路径组件
    std::cout << "根路径: " << full.root_path() << std::endl;
    std::cout << "根名: " << full.root_name() << std::endl;
    std::cout << "根目录: " << full.root_directory() << std::endl;
    std::cout << "相对路径: " << full.relative_path() << std::endl;
    std::cout << "父路径: " << full.parent_path() << std::endl;
    std::cout << "文件名: " << full.filename() << std::endl;
    std::cout << "主干: " << full.stem() << std::endl;
    std::cout << "扩展名: " << full.extension() << std::endl;

    return 0;
}
```

---

## 文件检查

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "test.txt";

    // 创建测试文件
    std::ofstream(p.string()) << "Hello, World!";

    std::cout << std::boolalpha;
    std::cout << "存在: " << fs::exists(p) << std::endl;
    std::cout << "是文件: " << fs::is_regular_file(p) << std::endl;
    std::cout << "是目录: " << fs::is_directory(p) << std::endl;
    std::cout << "是符号链接: " << fs::is_symlink(p) << std::endl;

    // 文件大小
    std::cout << "大小: " << fs::file_size(p) << " 字节" << std::endl;

    // 清理
    fs::remove(p);

    return 0;
}
```

---

## 目录遍历

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path dir = ".";

    std::cout << "当前目录内容:\n";
    for (const auto& entry : fs::directory_iterator(dir)) {
        std::cout << "  " << entry.path().filename() << std::endl;
    }

    return 0;
}
```

---

## 递归遍历

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path dir = ".";

    std::cout << "递归遍历:\n";
    for (const auto& entry : fs::recursive_directory_iterator(dir)) {
        std::cout << entry.path() << std::endl;
    }

    return 0;
}
```

---

## 创建目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path dir = "test_dir";
    fs::path nested = "test_dir/sub1/sub2";

    // 创建单个目录
    if (fs::create_directory(dir)) {
        std::cout << "创建目录: " << dir << std::endl;
    }

    // 创建嵌套目录
    if (fs::create_directories(nested)) {
        std::cout << "创建嵌套目录: " << nested << std::endl;
    }

    // 清理
    fs::remove_all(dir);

    return 0;
}
```

---

## 文件复制

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>

namespace fs = boost::filesystem;

int main() {
    fs::path source = "source.txt";
    fs::path dest = "dest.txt";

    // 创建源文件
    std::ofstream(source.string()) << "Hello, World!";

    // 复制文件
    fs::copy_file(source, dest);
    std::cout << "文件已复制: " << source << " -> " << dest << std::endl;

    // 验证
    std::cout << "源文件大小: " << fs::file_size(source) << std::endl;
    std::cout << "目标文件大小: " << fs::file_size(dest) << std::endl;

    // 清理
    fs::remove(source);
    fs::remove(dest);

    return 0;
}
```

---

## 文件重命名

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>

namespace fs = boost::filesystem;

int main() {
    fs::path old_name = "old.txt";
    fs::path new_name = "new.txt";

    // 创建文件
    std::ofstream(old_name.string()) << "Content";

    // 重命名
    fs::rename(old_name, new_name);
    std::cout << "重命名: " << old_name << " -> " << new_name << std::endl;

    std::cout << "旧文件存在: " << fs::exists(old_name) << std::endl;
    std::cout << "新文件存在: " << fs::exists(new_name) << std::endl;

    // 清理
    fs::remove(new_name);

    return 0;
}
```

---

## 查找文件

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <vector>

namespace fs = boost::filesystem;

std::vector<fs::path> find_files(const fs::path& dir, const std::string& ext) {
    std::vector<fs::path> result;

    for (const auto& entry : fs::recursive_directory_iterator(dir)) {
        if (entry.path().extension() == ext) {
            result.push_back(entry.path());
        }
    }

    return result;
}

int main() {
    // 查找所有 .txt 文件
    auto txt_files = find_files(".", ".txt");

    std::cout << "找到 " << txt_files.size() << " 个 .txt 文件:\n";
    for (const auto& file : txt_files) {
        std::cout << "  " << file << std::endl;
    }

    return 0;
}
```

---

## 文件时间

```cpp
#include <boost/filesystem.hpp>
#include <iostream>
#include <fstream>
#include <ctime>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "test.txt";

    // 创建文件
    std::ofstream(p.string()) << "Test";

    // 获取最后修改时间
    std::time_t t = fs::last_write_time(p);
    std::cout << "最后修改: " << std::ctime(&t);

    // 修改时间
    std::time_t new_time = std::time(nullptr) - 3600;  // 1小时前
    fs::last_write_time(p, new_time);

    t = fs::last_write_time(p);
    std::cout << "修改后: " << std::ctime(&t);

    // 清理
    fs::remove(p);

    return 0;
}
```

---

## 临时目录

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    // 获取临时目录
    fs::path temp_dir = fs::temp_directory_path();
    std::cout << "临时目录: " << temp_dir << std::endl;

    // 创建临时文件
    fs::path temp_file = temp_dir / "temp_file.txt";
    std::ofstream(temp_file.string()) << "Temporary data";

    std::cout << "临时文件: " << temp_file << std::endl;
    std::cout << "存在: " << fs::exists(temp_file) << std::endl;

    // 清理
    fs::remove(temp_file);

    return 0;
}
```

---

## 磁盘空间

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p = ".";

    fs::space_info si = fs::space(p);

    std::cout << "磁盘空间信息:\n";
    std::cout << "  总容量: " << si.capacity / (1024 * 1024 * 1024) << " GB\n";
    std::cout << "  可用空间: " << si.available / (1024 * 1024 * 1024) << " GB\n";
    std::cout << "  剩余空间: " << si.free / (1024 * 1024 * 1024) << " GB\n";

    return 0;
}
```

---

## 权限检查

```cpp
#include <boost/filesystem.hpp>
#include <iostream>

namespace fs = boost::filesystem;

int main() {
    fs::path p = "test.txt";

    // 创建文件
    std::ofstream(p.string()) << "Test";

    // 获取权限
    fs::perms perm = fs::status(p).permissions();

    std::cout << "文件权限:\n";
    std::cout << "  所有者可读: " << ((perm & fs::owner_read) != fs::no_perms) << std::endl;
    std::cout << "  所有者可写: " << ((perm & fs::owner_write) != fs::no_perms) << std::endl;
    std::cout << "  所有者可执行: " << ((perm & fs::owner_exe) != fs::no_perms) << std::endl;

    // 清理
    fs::remove(p);

    return 0;
}
```

---

## 参考资源

- [Boost.Filesystem 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/filesystem/doc/index.htm)
- [C++17 std::filesystem](https://en.cppreference.com/w/cpp/filesystem)

# Boost.Iostreams - IO流库

## 概述

Boost.Iostreams 提供可扩展的流处理框架，支持压缩、过滤等功能。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/zlib.hpp>
#include <boost/iostreams/copy.hpp>
#include <iostream>
#include <sstream>

int main() {
    namespace io = boost::iostreams;

    std::string data = "Hello, Boost.Iostreams! This is a test string.";

    // 压缩
    std::stringstream compressed;
    {
        io::filtering_ostream out;
        out.push(io::zlib_compressor());
        out.push(compressed);
        out << data;
    }

    std::cout << "原始大小: " << data.size() << " 字节" << std::endl;
    std::cout << "压缩后: " << compressed.str().size() << " 字节" << std::endl;

    // 解压
    std::string decompressed;
    {
        std::istringstream input(compressed.str());
        io::filtering_istream in;
        in.push(io::zlib_decompressor());
        in.push(input);

        std::stringstream ss;
        io::copy(in, ss);
        decompressed = ss.str();
    }

    std::cout << "解压后: " << decompressed << std::endl;

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_iostreams -lz`

---

## 文件压缩

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/gzip.hpp>
#include <boost/iostreams/copy.hpp>
#include <fstream>
#include <iostream>

namespace io = boost::iostreams;

int main() {
    // 创建原始文件
    {
        std::ofstream file("input.txt");
        file << "This is a test file.\n";
        file << "It will be compressed.\n";
        file << "Using Boost.Iostreams!\n";
    }

    // 压缩文件
    {
        std::ifstream input("input.txt", std::ios::binary);
        std::ofstream output("output.gz", std::ios::binary);

        io::filtering_ostream out;
        out.push(io::gzip_compressor());
        out.push(output);

        io::copy(input, out);
    }

    std::cout << "文件已压缩" << std::endl;

    // 解压文件
    {
        std::ifstream input("output.gz", std::ios::binary);
        std::ofstream output("decompressed.txt");

        io::filtering_istream in;
        in.push(io::gzip_decompressor());
        in.push(input);

        io::copy(in, output);
    }

    std::cout << "文件已解压" << std::endl;

    return 0;
}
```

---

## 自定义过滤器

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/concepts.hpp>
#include <iostream>
#include <sstream>

namespace io = boost::iostreams;

// 大写转换过滤器
class uppercase_filter : public io::input_filter {
public:
    template<typename Source>
    int get(Source& src) {
        int c = io::get(src);
        return c == EOF || c == io::WOULD_BLOCK ? c : std::toupper((unsigned char)c);
    }
};

int main() {
    std::string input = "Hello, World!";
    std::istringstream iss(input);

    io::filtering_istream in;
    in.push(uppercase_filter());
    in.push(iss);

    std::string result;
    std::getline(in, result);

    std::cout << "原始: " << input << std::endl;
    std::cout << "转换后: " << result << std::endl;

    return 0;
}
```

---

## 多个过滤器

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/counter.hpp>
#include <iostream>
#include <sstream>

namespace io = boost::iostreams;

int main() {
    std::string data = "Line 1\nLine 2\nLine 3\nLine 4\n";
    std::istringstream iss(data);

    io::filtering_istream in;
    io::counter counter;

    in.push(boost::ref(counter));
    in.push(iss);

    std::string line;
    while (std::getline(in, line)) {
        std::cout << line << std::endl;
    }

    std::cout << "\n统计:\n";
    std::cout << "字符数: " << counter.characters() << std::endl;
    std::cout << "行数: " << counter.lines() << std::endl;

    return 0;
}
```

---

## 内存映射文件

```cpp
#include <boost/iostreams/device/mapped_file.hpp>
#include <iostream>
#include <fstream>

namespace io = boost::iostreams;

int main() {
    // 创建测试文件
    {
        std::ofstream file("test.txt");
        file << "Memory mapped file test";
    }

    // 读取映射
    {
        io::mapped_file_source file("test.txt");

        if (file.is_open()) {
            const char* data = file.data();
            size_t size = file.size();

            std::cout << "文件内容: ";
            std::cout.write(data, size);
            std::cout << std::endl;
        }
    }

    // 写入映射
    {
        io::mapped_file file("test.txt", io::mapped_file::readwrite);

        if (file.is_open()) {
            char* data = file.data();
            data[0] = 'X';  // 修改第一个字符
        }
    }

    return 0;
}
```

---

## 缓冲设备

```cpp
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/stream.hpp>
#include <iostream>

namespace io = boost::iostreams;

int main() {
    char buffer[100];

    // 写入
    {
        io::stream<io::array_sink> out(buffer, sizeof(buffer));
        out << "Hello, " << 42 << " " << 3.14;
        out.flush();
    }

    std::cout << "缓冲区: " << buffer << std::endl;

    // 读取
    {
        io::stream<io::array_source> in(buffer, sizeof(buffer));
        std::string str;
        int num;
        double d;

        in >> str >> num >> d;

        std::cout << "读取: " << str << ", " << num << ", " << d << std::endl;
    }

    return 0;
}
```

---

## 行过滤

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/concepts.hpp>
#include <iostream>
#include <sstream>

namespace io = boost::iostreams;

// 行号过滤器
class line_number_filter : public io::output_filter {
private:
    int line_number_;

public:
    line_number_filter() : line_number_(1) {}

    template<typename Sink>
    bool put(Sink& dest, int c) {
        if (line_number_ == 1 || last_char_ == '\n') {
            std::string prefix = std::to_string(line_number_++) + ": ";
            for (char ch : prefix) {
                if (!io::put(dest, ch))
                    return false;
            }
        }

        last_char_ = c;
        return io::put(dest, c);
    }

private:
    char last_char_ = '\n';
};

int main() {
    std::ostringstream oss;

    io::filtering_ostream out;
    out.push(line_number_filter());
    out.push(oss);

    out << "First line\n";
    out << "Second line\n";
    out << "Third line\n";
    out.flush();

    std::cout << oss.str();

    return 0;
}
```

---

## 参考资源

- [Boost.Iostreams 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/iostreams/doc/index.html)

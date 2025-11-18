# Boost.Iostreams - I/O 流库

## 概述

Boost.Iostreams 提供用于创建流和流缓冲区的框架，支持过滤、压缩、加密等 I/O 操作。

**类型**: 需要编译链接的库

**链接库**: `-lboost_iostreams`

**可选依赖**: zlib (gzip 压缩), bzip2 (bzip2 压缩)

---

## 快速开始

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/gzip.hpp>
#include <boost/iostreams/copy.hpp>
#include <iostream>
#include <fstream>
#include <sstream>

namespace io = boost::iostreams;

int main() {
    // 创建过滤输出流
    io::filtering_ostream out;

    // 添加 gzip 压缩过滤器
    out.push(io::gzip_compressor());

    // 添加文件输出
    std::ofstream file("output.gz", std::ios::binary);
    out.push(file);

    // 写入数据（自动压缩）
    out << "Hello, Boost.Iostreams!" << std::endl;
    out << "This data will be compressed." << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_iostreams -lz -o example
```

---

## 文件压缩和解压

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/gzip.hpp>
#include <boost/iostreams/copy.hpp>
#include <iostream>
#include <fstream>

namespace io = boost::iostreams;

// 压缩文件
void compress_file(const std::string& input_file,
                  const std::string& output_file) {
    std::ifstream in(input_file, std::ios::binary);
    std::ofstream out(output_file, std::ios::binary);

    io::filtering_ostream filter_out;
    filter_out.push(io::gzip_compressor());
    filter_out.push(out);

    io::copy(in, filter_out);
}

// 解压文件
void decompress_file(const std::string& input_file,
                    const std::string& output_file) {
    std::ifstream in(input_file, std::ios::binary);
    std::ofstream out(output_file, std::ios::binary);

    io::filtering_istream filter_in;
    filter_in.push(io::gzip_decompressor());
    filter_in.push(in);

    io::copy(filter_in, out);
}

int main() {
    // 压缩
    compress_file("input.txt", "input.txt.gz");
    std::cout << "文件已压缩" << std::endl;

    // 解压
    decompress_file("input.txt.gz", "output.txt");
    std::cout << "文件已解压" << std::endl;

    return 0;
}
```

---

## Bzip2 压缩

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/bzip2.hpp>
#include <boost/iostreams/copy.hpp>
#include <iostream>
#include <fstream>
#include <sstream>

namespace io = boost::iostreams;

int main() {
    std::string data = "This is some data to compress with bzip2. "
                      "Bzip2 usually has better compression than gzip.";

    // 压缩
    std::stringstream compressed;
    {
        io::filtering_ostream out;
        out.push(io::bzip2_compressor());
        out.push(compressed);
        out << data;
    }

    std::cout << "原始大小: " << data.size() << " 字节" << std::endl;
    std::cout << "压缩后大小: " << compressed.str().size() << " 字节" << std::endl;

    // 解压
    std::string decompressed;
    {
        io::filtering_istream in;
        in.push(io::bzip2_decompressor());
        in.push(compressed);

        std::stringstream ss;
        io::copy(in, ss);
        decompressed = ss.str();
    }

    std::cout << "解压成功: " << (data == decompressed ? "是" : "否") << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_iostreams -lbz2 -o example
```

---

## 自定义过滤器

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/aggregate.hpp>
#include <boost/iostreams/concepts.hpp>
#include <boost/iostreams/operations.hpp>
#include <iostream>
#include <sstream>
#include <algorithm>

namespace io = boost::iostreams;

// 大写转换过滤器
class UppercaseFilter : public io::aggregate_filter<char> {
private:
    typedef io::aggregate_filter<char> base_type;

public:
    typedef base_type::char_type char_type;
    typedef base_type::category category;
    typedef base_type::vector_type vector_type;

private:
    void do_filter(const vector_type& src, vector_type& dest) override {
        dest.resize(src.size());
        std::transform(src.begin(), src.end(), dest.begin(), ::toupper);
    }
};

int main() {
    std::stringstream input("hello, world!");
    std::stringstream output;

    // 应用自定义过滤器
    io::filtering_istream in;
    in.push(UppercaseFilter());
    in.push(input);

    io::copy(in, output);

    std::cout << "输入: hello, world!" << std::endl;
    std::cout << "输出: " << output.str() << std::endl;

    return 0;
}
```

---

## 字符计数过滤器

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/counter.hpp>
#include <iostream>
#include <sstream>

namespace io = boost::iostreams;

int main() {
    std::stringstream ss;
    ss << "Hello, Boost.Iostreams!\n";
    ss << "This is a test.\n";

    // 创建带计数的输入流
    io::filtering_istream in;
    io::counter counter;
    in.push(boost::ref(counter));
    in.push(ss);

    // 读取所有内容
    std::string line;
    while (std::getline(in, line)) {
        std::cout << line << std::endl;
    }

    // 输出统计信息
    std::cout << "\n统计信息:" << std::endl;
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
#include <cstring>

namespace io = boost::iostreams;

int main() {
    const char* filename = "test_mapped.dat";

    // 创建并写入文件
    {
        io::mapped_file_params params;
        params.path = filename;
        params.new_file_size = 1024;
        params.mode = std::ios_base::out | std::ios_base::in;

        io::mapped_file file(params);

        if (file.is_open()) {
            char* data = file.data();
            std::strcpy(data, "Hello, Memory-Mapped File!");

            std::cout << "写入成功" << std::endl;
        }
    }

    // 读取文件
    {
        io::mapped_file_source file(filename);

        if (file.is_open()) {
            const char* data = file.data();
            std::cout << "读取: " << data << std::endl;
        }
    }

    return 0;
}
```

---

## 组合多个过滤器

```cpp
#include <boost/iostreams/filtering_stream.hpp>
#include <boost/iostreams/filter/gzip.hpp>
#include <boost/iostreams/filter/counter.hpp>
#include <boost/iostreams/copy.hpp>
#include <iostream>
#include <sstream>

namespace io = boost::iostreams;

int main() {
    std::string original_data = "This is test data for compression and counting.";

    std::stringstream compressed;

    // 组合计数器和压缩器
    io::filtering_ostream out;
    io::counter counter;

    out.push(boost::ref(counter));      // 先计数
    out.push(io::gzip_compressor());     // 再压缩
    out.push(compressed);

    // 写入数据
    out << original_data;
    out.flush();

    std::cout << "原始数据大小: " << counter.characters() << " 字节" << std::endl;
    std::cout << "压缩后大小: " << compressed.str().size() << " 字节" << std::endl;

    // 解压并验证
    std::stringstream decompressed;
    io::filtering_istream in;
    in.push(io::gzip_decompressor());
    in.push(compressed);

    io::copy(in, decompressed);

    std::cout << "解压后: " << decompressed.str() << std::endl;
    std::cout << "数据完整: "
              << (original_data == decompressed.str() ? "是" : "否") << std::endl;

    return 0;
}
```

---

## 流缓冲区设备

```cpp
#include <boost/iostreams/device/array.hpp>
#include <boost/iostreams/stream.hpp>
#include <iostream>
#include <cstring>

namespace io = boost::iostreams;

int main() {
    // 从数组读取
    char data[] = "Hello from array!";
    io::stream<io::array_source> in(data, std::strlen(data));

    std::string line;
    std::getline(in, line);
    std::cout << "读取: " << line << std::endl;

    // 写入到数组
    char buffer[100];
    io::stream<io::array_sink> out(buffer, sizeof(buffer));

    out << "Writing to array buffer";
    out.flush();

    std::cout << "写入: " << buffer << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Iostreams 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/iostreams/doc/index.html)
- [Boost.Iostreams 教程](https://www.boost.org/doc/libs/1_90_0/libs/iostreams/doc/tutorial/tutorial.html)

# Boost.Endian - 字节序处理库

## 概述

Boost.Endian 提供字节序转换和处理工具，用于在不同字节序系统之间传输数据。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::endian;

    uint32_t value = 0x12345678;

    std::cout << std::hex << std::setfill('0');
    std::cout << "原始值: 0x" << std::setw(8) << value << std::endl;

    // 大端序转换
    uint32_t big_endian = native_to_big(value);
    std::cout << "大端序: 0x" << std::setw(8) << big_endian << std::endl;

    // 小端序转换
    uint32_t little_endian = native_to_little(value);
    std::cout << "小端序: 0x" << std::setw(8) << little_endian << std::endl;

    // 字节反转
    uint32_t reversed = endian_reverse(value);
    std::cout << "反转: 0x" << std::setw(8) << reversed << std::endl;

    return 0;
}
```

---

## 字节序检测

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>

int main() {
    using namespace boost::endian;

    // 检测系统字节序
#ifdef BOOST_ENDIAN_BIG_BYTE
    std::cout << "系统字节序: 大端序 (Big Endian)" << std::endl;
#else
    std::cout << "系统字节序: 小端序 (Little Endian)" << std::endl;
#endif

    // 运行时检测
    uint32_t test = 0x01020304;
    uint8_t* bytes = reinterpret_cast<uint8_t*>(&test);

    if (bytes[0] == 0x01) {
        std::cout << "运行时检测: 大端序" << std::endl;
    } else {
        std::cout << "运行时检测: 小端序" << std::endl;
    }

    return 0;
}
```

---

## 条件转换

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::endian;

    uint32_t value = 0xAABBCCDD;

    std::cout << std::hex << std::setfill('0');

    // 只在需要时转换（目标是大端序）
    uint32_t as_big = conditional_reverse<order::big, order::native>(value);
    std::cout << "条件转换为大端: 0x" << std::setw(8) << as_big << std::endl;

    // 只在需要时转换（目标是小端序）
    uint32_t as_little = conditional_reverse<order::little, order::native>(value);
    std::cout << "条件转换为小端: 0x" << std::setw(8) << as_little << std::endl;

    return 0;
}
```

---

## 缓冲区加载和存储

```cpp
#include <boost/endian/buffers.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::endian;

    // 创建大端序32位整数缓冲区
    big_int32_buf_t big_buf;
    big_buf = 0x12345678;

    std::cout << std::hex << std::setfill('0');
    std::cout << "大端序缓冲: 0x" << std::setw(8) << big_buf.value() << std::endl;

    // 创建小端序32位整数缓冲区
    little_int32_buf_t little_buf;
    little_buf = 0x12345678;

    std::cout << "小端序缓冲: 0x" << std::setw(8) << little_buf.value() << std::endl;

    // 查看字节布局
    uint8_t* bytes = reinterpret_cast<uint8_t*>(&big_buf);
    std::cout << "大端字节序列: ";
    for (int i = 0; i < 4; ++i) {
        std::cout << std::setw(2) << static_cast<int>(bytes[i]) << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 网络字节序

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <cstdint>

int main() {
    using namespace boost::endian;

    // 网络字节序是大端序
    uint16_t port = 8080;
    uint32_t ip = 0xC0A80001;  // 192.168.0.1

    // 转换为网络字节序
    uint16_t network_port = native_to_big(port);
    uint32_t network_ip = native_to_big(ip);

    std::cout << "主机端口: " << port << std::endl;
    std::cout << "网络端口: " << network_port << std::endl;

    // 从网络字节序转换回来
    uint16_t host_port = big_to_native(network_port);
    uint32_t host_ip = big_to_native(network_ip);

    std::cout << "\n恢复主机端口: " << host_port << std::endl;

    return 0;
}
```

---

## 结构体字节序

```cpp
#include <boost/endian/arithmetic.hpp>
#include <iostream>
#include <cstring>

using namespace boost::endian;

// 网络协议头（大端序）
struct PacketHeader {
    big_uint16_t magic;
    big_uint16_t length;
    big_uint32_t sequence;
    big_uint32_t timestamp;
};

int main() {
    PacketHeader header;
    header.magic = 0xCAFE;
    header.length = 1024;
    header.sequence = 12345;
    header.timestamp = 1234567890;

    std::cout << "Magic: 0x" << std::hex << header.magic.value() << std::endl;
    std::cout << "Length: " << std::dec << header.length.value() << std::endl;
    std::cout << "Sequence: " << header.sequence.value() << std::endl;

    // 序列化到字节数组
    uint8_t buffer[sizeof(PacketHeader)];
    std::memcpy(buffer, &header, sizeof(header));

    std::cout << "\n序列化字节: ";
    for (size_t i = 0; i < sizeof(buffer); ++i) {
        std::cout << std::hex << std::setfill('0') << std::setw(2)
                  << static_cast<int>(buffer[i]) << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 原地转换

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::endian;

    std::vector<uint32_t> data = {0x11111111, 0x22222222, 0x33333333};

    std::cout << "原始数据:\n";
    for (auto val : data) {
        std::cout << std::hex << "0x" << val << " ";
    }
    std::cout << std::endl;

    // 原地反转字节序
    for (auto& val : data) {
        endian_reverse_inplace(val);
    }

    std::cout << "\n反转后:\n";
    for (auto val : data) {
        std::cout << std::hex << "0x" << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 文件I/O

```cpp
#include <boost/endian/arithmetic.hpp>
#include <iostream>
#include <fstream>

using namespace boost::endian;

struct FileHeader {
    big_uint32_t magic;
    big_uint32_t version;
    big_uint64_t file_size;
};

int main() {
    const char* filename = "data.bin";

    // 写入文件
    {
        std::ofstream file(filename, std::ios::binary);

        FileHeader header;
        header.magic = 0xDEADBEEF;
        header.version = 1;
        header.file_size = 1024;

        file.write(reinterpret_cast<const char*>(&header), sizeof(header));

        std::cout << "写入文件头" << std::endl;
    }

    // 读取文件
    {
        std::ifstream file(filename, std::ios::binary);

        FileHeader header;
        file.read(reinterpret_cast<char*>(&header), sizeof(header));

        std::cout << "\n读取文件头:" << std::endl;
        std::cout << "Magic: 0x" << std::hex << header.magic.value() << std::endl;
        std::cout << "Version: " << std::dec << header.version.value() << std::endl;
        std::cout << "File Size: " << header.file_size.value() << std::endl;
    }

    return 0;
}
```

---

## 位域

```cpp
#include <boost/endian/arithmetic.hpp>
#include <iostream>

using namespace boost::endian;

// 标志位（大端序）
struct Flags {
    unsigned int flag1 : 1;
    unsigned int flag2 : 1;
    unsigned int reserved : 6;
    unsigned int value : 8;
};

int main() {
    big_uint16_t packed = 0;

    // 手动打包位
    packed = (1 << 15) | (0 << 14) | (0x42 << 0);

    std::cout << "打包值: 0x" << std::hex << packed.value() << std::endl;

    // 解包
    bool flag1 = (packed.value() >> 15) & 1;
    bool flag2 = (packed.value() >> 14) & 1;
    uint8_t value = packed.value() & 0xFF;

    std::cout << "Flag1: " << flag1 << std::endl;
    std::cout << "Flag2: " << flag2 << std::endl;
    std::cout << "Value: 0x" << std::hex << static_cast<int>(value) << std::endl;

    return 0;
}
```

---

## 数组转换

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::endian;

    std::vector<uint16_t> data = {0x0102, 0x0304, 0x0506, 0x0708};

    std::cout << "原始数据: ";
    for (auto val : data) {
        std::cout << std::hex << "0x" << val << " ";
    }
    std::cout << std::endl;

    // 批量转换为大端序
    std::vector<uint16_t> big_endian(data.size());
    for (size_t i = 0; i < data.size(); ++i) {
        big_endian[i] = native_to_big(data[i]);
    }

    std::cout << "大端序: ";
    for (auto val : big_endian) {
        std::cout << std::hex << "0x" << val << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 性能考虑

```cpp
#include <boost/endian/conversion.hpp>
#include <iostream>
#include <chrono>
#include <vector>

int main() {
    using namespace boost::endian;

    const int size = 1000000;
    std::vector<uint32_t> data(size);

    // 填充数据
    for (int i = 0; i < size; ++i) {
        data[i] = i;
    }

    // 测试转换性能
    auto start = std::chrono::high_resolution_clock::now();

    for (auto& val : data) {
        val = native_to_big(val);
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "转换 " << size << " 个整数耗时: "
              << duration.count() << " 毫秒" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Endian 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/endian/doc/html/index.html)
- [字节序详解](https://en.wikipedia.org/wiki/Endianness)

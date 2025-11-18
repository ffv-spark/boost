# Boost.CRC - 循环冗余校验库

## 概述

Boost.CRC 提供循环冗余校验（CRC）算法的实现，用于数据完整性检查。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "Hello, World!";

    // 计算 CRC-32
    boost::crc_32_type crc;
    crc.process_bytes(data.data(), data.length());

    std::cout << "数据: " << data << std::endl;
    std::cout << "CRC-32: 0x" << std::hex << crc.checksum() << std::endl;

    return 0;
}
```

---

## CRC-16

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "Test data";

    // CRC-16
    boost::crc_16_type crc16;
    crc16.process_bytes(data.data(), data.length());

    std::cout << "数据: " << data << std::endl;
    std::cout << "CRC-16: 0x" << std::hex << crc16.checksum() << std::endl;

    return 0;
}
```

---

## CRC-32

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "The quick brown fox jumps over the lazy dog";

    // CRC-32
    boost::crc_32_type crc32;
    crc32.process_bytes(data.data(), data.length());

    std::cout << "数据: " << data << std::endl;
    std::cout << "CRC-32: 0x" << std::hex << crc32.checksum() << std::endl;

    return 0;
}
```

---

## CCITT CRC

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "CCITT test";

    // CCITT CRC
    boost::crc_ccitt_type crc;
    crc.process_bytes(data.data(), data.length());

    std::cout << "数据: " << data << std::endl;
    std::cout << "CRC-CCITT: 0x" << std::hex << crc.checksum() << std::endl;

    return 0;
}
```

---

## 文件校验和

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <fstream>
#include <vector>

unsigned int calculate_file_crc(const std::string& filename) {
    std::ifstream file(filename, std::ios::binary);
    if (!file) {
        throw std::runtime_error("无法打开文件");
    }

    boost::crc_32_type crc;
    const size_t buffer_size = 4096;
    std::vector<char> buffer(buffer_size);

    while (file.read(buffer.data(), buffer_size) || file.gcount() > 0) {
        crc.process_bytes(buffer.data(), file.gcount());
    }

    return crc.checksum();
}

int main() {
    try {
        // 创建测试文件
        {
            std::ofstream file("test.txt");
            file << "This is a test file for CRC calculation.";
        }

        unsigned int crc = calculate_file_crc("test.txt");
        std::cout << "文件 CRC-32: 0x" << std::hex << crc << std::endl;

    } catch (const std::exception& e) {
        std::cerr << "错误: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 增量计算

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    boost::crc_32_type crc;

    // 分块处理数据
    std::string part1 = "Hello, ";
    std::string part2 = "World!";

    crc.process_bytes(part1.data(), part1.length());
    std::cout << "处理第一部分后: 0x" << std::hex << crc.checksum() << std::endl;

    crc.process_bytes(part2.data(), part2.length());
    std::cout << "处理第二部分后: 0x" << std::hex << crc.checksum() << std::endl;

    // 一次性计算
    boost::crc_32_type crc2;
    std::string full = part1 + part2;
    crc2.process_bytes(full.data(), full.length());
    std::cout << "一次性计算: 0x" << std::hex << crc2.checksum() << std::endl;

    std::cout << "\n结果相同: " << (crc.checksum() == crc2.checksum()) << std::endl;

    return 0;
}
```

---

## 数据完整性检查

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

struct Packet {
    std::string data;
    unsigned int crc;
};

Packet create_packet(const std::string& data) {
    boost::crc_32_type crc;
    crc.process_bytes(data.data(), data.length());

    return {data, crc.checksum()};
}

bool verify_packet(const Packet& packet) {
    boost::crc_32_type crc;
    crc.process_bytes(packet.data.data(), packet.data.length());

    return crc.checksum() == packet.crc;
}

int main() {
    // 创建数据包
    Packet p1 = create_packet("Important data");

    std::cout << "数据: " << p1.data << std::endl;
    std::cout << "CRC: 0x" << std::hex << p1.crc << std::endl;
    std::cout << "验证: " << (verify_packet(p1) ? "通过" : "失败") << std::endl;

    // 模拟数据损坏
    Packet p2 = p1;
    p2.data[0] = 'X';

    std::cout << "\n损坏的数据: " << p2.data << std::endl;
    std::cout << "验证: " << (verify_packet(p2) ? "通过" : "失败") << std::endl;

    return 0;
}
```

---

## 自定义 CRC

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    // 自定义 CRC 参数
    // CRC-8: 多项式 0x07，初始值 0x00
    boost::crc_optimal<8, 0x07, 0, 0, false, false> crc8;

    std::string data = "Custom CRC";
    crc8.process_bytes(data.data(), data.length());

    std::cout << "数据: " << data << std::endl;
    std::cout << "CRC-8: 0x" << std::hex << (int)crc8.checksum() << std::endl;

    return 0;
}
```

---

## 网络协议校验

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <vector>
#include <cstring>

struct NetworkFrame {
    char header[4];
    char data[256];
    unsigned short length;
    unsigned int crc;
};

void compute_frame_crc(NetworkFrame& frame) {
    boost::crc_32_type crc;

    // 计算头部和数据的 CRC
    crc.process_bytes(frame.header, sizeof(frame.header));
    crc.process_bytes(frame.data, frame.length);

    frame.crc = crc.checksum();
}

bool verify_frame(const NetworkFrame& frame) {
    boost::crc_32_type crc;

    crc.process_bytes(frame.header, sizeof(frame.header));
    crc.process_bytes(frame.data, frame.length);

    return crc.checksum() == frame.crc;
}

int main() {
    NetworkFrame frame;
    std::memcpy(frame.header, "HEAD", 4);
    std::memcpy(frame.data, "Network payload", 15);
    frame.length = 15;

    compute_frame_crc(frame);

    std::cout << "帧头: " << std::string(frame.header, 4) << std::endl;
    std::cout << "数据: " << std::string(frame.data, frame.length) << std::endl;
    std::cout << "CRC: 0x" << std::hex << frame.crc << std::endl;
    std::cout << "验证: " << (verify_frame(frame) ? "通过" : "失败") << std::endl;

    return 0;
}
```

---

## 性能对比

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <vector>
#include <chrono>

int main() {
    const size_t data_size = 1000000;
    std::vector<char> data(data_size, 'A');

    // CRC-16
    auto start1 = std::chrono::high_resolution_clock::now();
    boost::crc_16_type crc16;
    crc16.process_bytes(data.data(), data.size());
    auto end1 = std::chrono::high_resolution_clock::now();

    // CRC-32
    auto start2 = std::chrono::high_resolution_clock::now();
    boost::crc_32_type crc32;
    crc32.process_bytes(data.data(), data.size());
    auto end2 = std::chrono::high_resolution_clock::now();

    auto duration1 = std::chrono::duration_cast<std::chrono::microseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::microseconds>(end2 - start2);

    std::cout << "处理 " << data_size << " 字节:\n";
    std::cout << "CRC-16: " << duration1.count() << " 微秒" << std::endl;
    std::cout << "CRC-32: " << duration2.count() << " 微秒" << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.CRC 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/crc/crc.html)
- [CRC 算法](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)

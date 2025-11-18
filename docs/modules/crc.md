# Boost.CRC - CRC 校验库

## 概述

Boost.CRC 提供循环冗余校验（CRC）计算功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "Hello, World!";

    // CRC-32
    boost::crc_32_type crc32;
    crc32.process_bytes(data.data(), data.size());

    std::cout << "CRC-32: 0x" << std::hex << crc32.checksum() << std::endl;

    return 0;
}
```

---

## 常用 CRC 类型

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>

int main() {
    std::string data = "Test data";

    // CRC-16
    boost::crc_16_type crc16;
    crc16.process_bytes(data.data(), data.size());
    std::cout << "CRC-16: 0x" << std::hex << crc16.checksum() << std::endl;

    // CRC-32
    boost::crc_32_type crc32;
    crc32.process_bytes(data.data(), data.size());
    std::cout << "CRC-32: 0x" << std::hex << crc32.checksum() << std::endl;

    // CCITT CRC
    boost::crc_ccitt_type crc_ccitt;
    crc_ccitt.process_bytes(data.data(), data.size());
    std::cout << "CRC-CCITT: 0x" << std::hex << crc_ccitt.checksum() << std::endl;

    return 0;
}
```

---

## 文件校验

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <fstream>

uint32_t calculate_file_crc32(const std::string& filename) {
    boost::crc_32_type crc;
    std::ifstream file(filename, std::ios::binary);

    if (!file) {
        throw std::runtime_error("Cannot open file");
    }

    char buffer[4096];
    while (file.read(buffer, sizeof(buffer))) {
        crc.process_bytes(buffer, file.gcount());
    }

    // 处理剩余字节
    if (file.gcount() > 0) {
        crc.process_bytes(buffer, file.gcount());
    }

    return crc.checksum();
}

int main() {
    try {
        uint32_t checksum = calculate_file_crc32("test.txt");
        std::cout << "File CRC-32: 0x" << std::hex << checksum << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }

    return 0;
}
```

---

## 数据完整性检查

```cpp
#include <boost/crc.hpp>
#include <iostream>
#include <string>
#include <vector>

struct Packet {
    std::vector<char> data;
    uint32_t checksum;
};

Packet create_packet(const std::string& message) {
    Packet packet;
    packet.data.assign(message.begin(), message.end());

    boost::crc_32_type crc;
    crc.process_bytes(packet.data.data(), packet.data.size());
    packet.checksum = crc.checksum();

    return packet;
}

bool verify_packet(const Packet& packet) {
    boost::crc_32_type crc;
    crc.process_bytes(packet.data.data(), packet.data.size());

    return crc.checksum() == packet.checksum;
}

int main() {
    Packet packet = create_packet("Important data");

    std::cout << "Packet checksum: 0x" << std::hex << packet.checksum << std::endl;

    if (verify_packet(packet)) {
        std::cout << "Packet is valid" << std::endl;
    } else {
        std::cout << "Packet is corrupted!" << std::endl;
    }

    // 模拟数据损坏
    packet.data[0] = 'X';

    if (verify_packet(packet)) {
        std::cout << "Packet is valid" << std::endl;
    } else {
        std::cout << "Packet is corrupted!" << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.CRC 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/crc/crc.html)

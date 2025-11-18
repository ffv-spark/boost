# Boost.Nowide - 跨平台Unicode库

## 概述

Boost.Nowide 提供跨平台的Unicode支持，简化Windows和POSIX系统上的文本处理。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/nowide/iostream.hpp>
#include <boost/nowide/fstream.hpp>

int main() {
    // 使用UTF-8的cout
    boost::nowide::cout << "你好，世界！" << std::endl;
    
    // UTF-8文件操作
    boost::nowide::ofstream file("test.txt");
    file << "UTF-8文本内容" << std::endl;
    file.close();
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_nowide`

---

## 参考资源

- [Boost.Nowide 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/nowide/doc/html/index.html)

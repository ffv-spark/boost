# Boost.DLL - 动态库加载

## 概述

Boost.DLL 提供跨平台的动态库（.so/.dll）加载和符号导入功能。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/dll/import.hpp>
#include <boost/dll/shared_library.hpp>
#include <iostream>

namespace dll = boost::dll;

int main() {
    // 加载动态库
    dll::shared_library lib("mylib.so");  // Linux: .so, Windows: .dll
    
    if (lib.has("my_function")) {
        // 导入函数
        auto func = lib.get<int(int, int)>("my_function");
        
        int result = func(3, 4);
        std::cout << "结果: " << result << std::endl;
    }
    
    return 0;
}
```

---

## 导入函数

```cpp
#include <boost/dll/import.hpp>
#include <iostream>
#include <boost/function.hpp>

int main() {
    namespace dll = boost::dll;
    
    // 导入函数（自动管理库生命周期）
    auto my_func = dll::import_alias<int(int, int)>(
        "mylib.so",
        "my_function"
    );
    
    std::cout << "调用函数: " << my_func(10, 20) << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.DLL 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_dll.html)

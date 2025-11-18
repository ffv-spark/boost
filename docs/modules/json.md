# Boost.JSON - JSON库

## 概述

Boost.JSON 提供快速、现代的JSON解析和序列化支持。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/json.hpp>
#include <iostream>

namespace json = boost::json;

int main() {
    // 创建JSON对象
    json::object obj;
    obj["name"] = "Alice";
    obj["age"] = 30;
    obj["city"] = "Beijing";
    
    // 转换为字符串
    std::string json_str = json::serialize(obj);
    std::cout << "JSON: " << json_str << std::endl;
    
    // 解析JSON
    json::value parsed = json::parse(json_str);
    std::cout << "姓名: " << parsed.at("name").as_string() << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_json`

---

## 参考资源

- [Boost.JSON 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/json/doc/html/index.html)

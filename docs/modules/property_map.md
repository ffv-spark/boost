# Boost.PropertyMap - 属性映射库

## 概述

Boost.PropertyMap 提供键值对抽象，常用于图算法中存储顶点和边的属性。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/property_map/property_map.hpp>
#include <iostream>
#include <map>

int main() {
    using namespace boost;
    
    std::map<int, std::string> name_map;
    name_map[0] = "Alice";
    name_map[1] = "Bob";
    name_map[2] = "Charlie";
    
    associative_property_map<std::map<int, std::string>> pmap(name_map);
    
    std::cout << "ID 0: " << get(pmap, 0) << std::endl;
    std::cout << "ID 1: " << get(pmap, 1) << std::endl;
    
    put(pmap, 3, "David");
    std::cout << "ID 3: " << get(pmap, 3) << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.PropertyMap 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/property_map/doc/property_map.html)

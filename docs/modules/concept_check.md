# Boost.ConceptCheck - 概念检查库

## 概述

Boost.ConceptCheck 提供编译期概念验证，确保模板参数满足特定要求。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/concept_check.hpp>
#include <iostream>
#include <vector>

template <typename T>
void sort_container(T& container) {
    BOOST_CONCEPT_ASSERT((boost::RandomAccessContainer<T>));
    
    std::sort(container.begin(), container.end());
}

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9};
    sort_container(vec);  // OK
    
    // std::list<int> lst = {5, 2, 8, 1, 9};
    // sort_container(lst);  // 编译错误：list不是RandomAccessContainer
    
    return 0;
}
```

---

## 参考资源

- [Boost.ConceptCheck 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/concept_check/concept_check.htm)

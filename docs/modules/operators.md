# Boost.Operators - 运算符辅助库

## 概述

Boost.Operators 提供运算符重载辅助工具，简化运算符定义。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/operators.hpp>
#include <iostream>

class Point : boost::equality_comparable<Point>,
              boost::less_than_comparable<Point> {
public:
    int x, y;
    
    Point(int x_, int y_) : x(x_), y(y_) {}
    
    // 只需定义 == 和 <，其他比较运算符自动生成
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
    
    bool operator<(const Point& other) const {
        if (x != other.x) return x < other.x;
        return y < other.y;
    }
};

int main() {
    Point p1(1, 2), p2(3, 4);
    
    std::cout << std::boolalpha;
    std::cout << "p1 == p2: " << (p1 == p2) << std::endl;
    std::cout << "p1 != p2: " << (p1 != p2) << std::endl;  // 自动生成
    std::cout << "p1 < p2: " << (p1 < p2) << std::endl;
    std::cout << "p1 > p2: " << (p1 > p2) << std::endl;    // 自动生成
    
    return 0;
}
```

---

## 参考资源

- [Boost.Operators 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/utility/operators.htm)

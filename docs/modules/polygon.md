# Boost.Polygon - 2D多边形库

## 概述

Boost.Polygon 提供2D多边形操作，包括布尔运算、Voronoi图等。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/polygon/polygon.hpp>
#include <iostream>
#include <vector>

namespace gtl = boost::polygon;
using namespace boost::polygon::operators;

int main() {
    typedef gtl::polygon_data<int> Polygon;
    typedef gtl::point_data<int> Point;
    
    // 创建多边形
    std::vector<Point> pts;
    pts.push_back(Point(0, 0));
    pts.push_back(Point(10, 0));
    pts.push_back(Point(10, 10));
    pts.push_back(Point(0, 10));
    
    Polygon poly;
    gtl::set_points(poly, pts.begin(), pts.end());
    
    // 计算面积
    std::cout << "面积: " << gtl::area(poly) << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Polygon 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/polygon/doc/index.htm)

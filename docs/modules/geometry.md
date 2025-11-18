# Boost.Geometry - 计算几何库

## 概述

Boost.Geometry 提供几何算法和数据结构，支持点、线、多边形等几何对象。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> point;
    typedef bg::model::polygon<point> polygon;
    
    // 创建多边形
    polygon poly;
    bg::read_wkt("POLYGON((0 0, 0 10, 10 10, 10 0, 0 0))", poly);
    
    // 计算面积
    double area = bg::area(poly);
    std::cout << "面积: " << area << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.Geometry 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/geometry/doc/html/index.html)

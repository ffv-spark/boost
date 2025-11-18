# Boost.Geometry - 几何算法库

## 概述

Boost.Geometry 提供用于 2D 和 3D 几何计算的数据结构和算法，支持点、线、多边形等几何对象的操作。

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
    // 定义 2D 点
    typedef bg::model::d2::point_xy<double> Point;

    Point p1(0, 0);
    Point p2(3, 4);

    // 计算两点之间的距离
    double dist = bg::distance(p1, p2);
    std::cout << "距离: " << dist << std::endl;

    return 0;
}
```

---

## 点和距离计算

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;

    Point p1(0, 0);
    Point p2(3, 4);
    Point p3(6, 8);

    // 欧几里得距离
    std::cout << "p1 到 p2 的距离: " << bg::distance(p1, p2) << std::endl;
    std::cout << "p2 到 p3 的距离: " << bg::distance(p2, p3) << std::endl;

    // 获取坐标
    std::cout << "p2 坐标: (" << bg::get<0>(p2) << ", "
              << bg::get<1>(p2) << ")" << std::endl;

    // 设置坐标
    Point p4;
    bg::set<0>(p4, 10.0);
    bg::set<1>(p4, 20.0);
    std::cout << "p4 坐标: (" << bg::get<0>(p4) << ", "
              << bg::get<1>(p4) << ")" << std::endl;

    return 0;
}
```

---

## 线段和折线

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/linestring.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::linestring<Point> Linestring;

    // 创建折线
    Linestring line;
    bg::append(line, Point(0, 0));
    bg::append(line, Point(1, 1));
    bg::append(line, Point(2, 0));
    bg::append(line, Point(3, 1));

    // 计算长度
    double length = bg::length(line);
    std::cout << "折线长度: " << length << std::endl;

    // 输出点数
    std::cout << "点数: " << line.size() << std::endl;

    // 遍历点
    std::cout << "折线上的点:\n";
    for (const auto& p : line) {
        std::cout << "  (" << bg::get<0>(p) << ", "
                  << bg::get<1>(p) << ")\n";
    }

    return 0;
}
```

---

## 多边形和面积

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::polygon<Point> Polygon;

    // 创建矩形
    Polygon rect;
    bg::append(rect.outer(), Point(0, 0));
    bg::append(rect.outer(), Point(0, 4));
    bg::append(rect.outer(), Point(3, 4));
    bg::append(rect.outer(), Point(3, 0));
    bg::append(rect.outer(), Point(0, 0));  // 闭合多边形

    // 计算面积
    double area = bg::area(rect);
    std::cout << "矩形面积: " << area << std::endl;

    // 计算周长
    double perimeter = bg::perimeter(rect);
    std::cout << "矩形周长: " << perimeter << std::endl;

    // 计算中心点
    Point centroid;
    bg::centroid(rect, centroid);
    std::cout << "中心点: (" << bg::get<0>(centroid) << ", "
              << bg::get<1>(centroid) << ")\n";

    return 0;
}
```

---

## 空间关系判断

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <boost/geometry/geometries/box.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::polygon<Point> Polygon;
    typedef bg::model::box<Point> Box;

    // 创建多边形
    Polygon poly;
    bg::append(poly.outer(), Point(0, 0));
    bg::append(poly.outer(), Point(0, 5));
    bg::append(poly.outer(), Point(5, 5));
    bg::append(poly.outer(), Point(5, 0));
    bg::append(poly.outer(), Point(0, 0));

    // 测试点是否在多边形内
    Point p1(2, 2);
    Point p2(10, 10);

    std::cout << "点 (2,2) 在多边形内: "
              << (bg::within(p1, poly) ? "是" : "否") << std::endl;
    std::cout << "点 (10,10) 在多边形内: "
              << (bg::within(p2, poly) ? "是" : "否") << std::endl;

    // 创建矩形框
    Box box(Point(1, 1), Point(3, 3));

    // 测试框是否在多边形内
    std::cout << "矩形框在多边形内: "
              << (bg::within(box, poly) ? "是" : "否") << std::endl;

    // 测试是否相交
    Box box2(Point(4, 4), Point(6, 6));
    std::cout << "矩形框2与多边形相交: "
              << (bg::intersects(box2, poly) ? "是" : "否") << std::endl;

    return 0;
}
```

---

## 缓冲区（Buffer）

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <boost/geometry/strategies/buffer.hpp>
#include <iostream>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::polygon<Point> Polygon;

    // 创建一个点
    Point p(5, 5);

    // 缓冲策略
    bg::strategy::buffer::distance_symmetric<double> distance_strategy(2.0);
    bg::strategy::buffer::join_round join_strategy(32);
    bg::strategy::buffer::end_round end_strategy(32);
    bg::strategy::buffer::point_circle point_strategy(32);
    bg::strategy::buffer::side_straight side_strategy;

    // 创建缓冲区（圆形）
    bg::model::multi_polygon<Polygon> result;
    bg::buffer(p, result,
               distance_strategy,
               side_strategy,
               join_strategy,
               end_strategy,
               point_strategy);

    // 输出结果
    if (!result.empty()) {
        std::cout << "缓冲区面积: " << bg::area(result[0]) << std::endl;
    }

    return 0;
}
```

---

## 凸包（Convex Hull）

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <iostream>
#include <vector>

namespace bg = boost::geometry;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::polygon<Point> Polygon;

    // 创建点集
    std::vector<Point> points = {
        Point(0, 0),
        Point(1, 1),
        Point(2, 0.5),
        Point(1.5, 1.5),
        Point(0.5, 1),
        Point(2, 2)
    };

    // 计算凸包
    Polygon hull;
    bg::convex_hull(points, hull);

    // 输出凸包
    std::cout << "凸包顶点数: " << hull.outer().size() - 1 << std::endl;
    std::cout << "凸包面积: " << bg::area(hull) << std::endl;

    std::cout << "凸包顶点:\n";
    for (const auto& p : hull.outer()) {
        std::cout << "  (" << bg::get<0>(p) << ", "
                  << bg::get<1>(p) << ")\n";
    }

    return 0;
}
```

---

## 几何变换

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point_xy.hpp>
#include <boost/geometry/geometries/polygon.hpp>
#include <iostream>

namespace bg = boost::geometry;
namespace trans = boost::geometry::strategy::transform;

int main() {
    typedef bg::model::d2::point_xy<double> Point;
    typedef bg::model::polygon<Point> Polygon;

    // 创建三角形
    Polygon triangle;
    bg::append(triangle.outer(), Point(0, 0));
    bg::append(triangle.outer(), Point(1, 0));
    bg::append(triangle.outer(), Point(0.5, 1));
    bg::append(triangle.outer(), Point(0, 0));

    std::cout << "原始三角形面积: " << bg::area(triangle) << std::endl;

    // 平移
    Polygon translated;
    trans::translate_transformer<double, 2, 2> translate(3.0, 2.0);
    bg::transform(triangle, translated, translate);

    std::cout << "平移后面积: " << bg::area(translated) << std::endl;

    // 缩放
    Polygon scaled;
    trans::scale_transformer<double, 2, 2> scale(2.0, 2.0);
    bg::transform(triangle, scaled, scale);

    std::cout << "缩放后面积: " << bg::area(scaled) << std::endl;

    return 0;
}
```

---

## 空间索引

```cpp
#include <boost/geometry.hpp>
#include <boost/geometry/geometries/point.hpp>
#include <boost/geometry/geometries/box.hpp>
#include <boost/geometry/index/rtree.hpp>
#include <iostream>
#include <vector>

namespace bg = boost::geometry;
namespace bgi = boost::geometry::index;

int main() {
    typedef bg::model::point<double, 2, bg::cs::cartesian> Point;
    typedef bg::model::box<Point> Box;
    typedef std::pair<Point, int> Value;

    // 创建 R-tree 空间索引
    bgi::rtree<Value, bgi::quadratic<16>> rtree;

    // 插入点
    for (int i = 0; i < 10; ++i) {
        for (int j = 0; j < 10; ++j) {
            Point p(i, j);
            rtree.insert(std::make_pair(p, i * 10 + j));
        }
    }

    std::cout << "R-tree 包含 " << rtree.size() << " 个点\n";

    // 查询范围内的点
    Box query_box(Point(2, 2), Point(5, 5));
    std::vector<Value> result;

    rtree.query(bgi::within(query_box), std::back_inserter(result));

    std::cout << "范围查询结果 (" << result.size() << " 个点):\n";
    for (const auto& v : result) {
        std::cout << "  点 (" << bg::get<0>(v.first) << ", "
                  << bg::get<1>(v.first) << "), ID: " << v.second << std::endl;
    }

    // K 近邻查询
    Point query_point(5, 5);
    std::vector<Value> nearest;

    rtree.query(bgi::nearest(query_point, 3), std::back_inserter(nearest));

    std::cout << "\n最近的 3 个点:\n";
    for (const auto& v : nearest) {
        std::cout << "  点 (" << bg::get<0>(v.first) << ", "
                  << bg::get<1>(v.first) << "), ID: " << v.second << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Geometry 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/geometry/doc/html/index.html)
- [Boost.Geometry 快速入门](https://www.boost.org/doc/libs/1_90_0/libs/geometry/doc/html/geometry/quickstart.html)

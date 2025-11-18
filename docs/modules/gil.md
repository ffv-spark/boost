# Boost.GIL - 通用图像库

## 概述

Boost.GIL (Generic Image Library) 提供通用的图像表示和算法，支持多种像素格式。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/gil.hpp>
#include <boost/gil/extension/io/jpeg.hpp>
#include <iostream>

namespace gil = boost::gil;

int main() {
    // 创建图像
    gil::rgb8_image_t img(640, 480);
    
    // 填充颜色
    gil::fill_pixels(view(img), gil::rgb8_pixel_t(255, 0, 0));  // 红色
    
    std::cout << "创建了 " << img.width() << "x" << img.height() << " 的图像" << std::endl;
    
    // 保存为JPEG（需要libjpeg）
    // gil::write_view("output.jpg", const_view(img), gil::jpeg_tag());
    
    return 0;
}
```

---

## 像素操作

```cpp
#include <boost/gil.hpp>
#include <iostream>

namespace gil = boost::gil;

int main() {
    gil::rgb8_image_t img(100, 100);
    auto v = view(img);
    
    // 设置单个像素
    v(0, 0) = gil::rgb8_pixel_t(255, 0, 0);      // 红色
    v(1, 1) = gil::rgb8_pixel_t(0, 255, 0);      // 绿色
    v(2, 2) = gil::rgb8_pixel_t(0, 0, 255);      // 蓝色
    
    // 读取像素
    auto pixel = v(0, 0);
    std::cout << "像素(0,0): R=" << (int)pixel[0] 
              << " G=" << (int)pixel[1] 
              << " B=" << (int)pixel[2] << std::endl;
    
    return 0;
}
```

---

## 图像视图

```cpp
#include <boost/gil.hpp>
#include <iostream>

namespace gil = boost::gil;

int main() {
    gil::rgb8_image_t img(200, 200);
    
    // 创建子视图
    auto sub_view = gil::subimage_view(view(img), 50, 50, 100, 100);
    
    // 填充子视图
    gil::fill_pixels(sub_view, gil::rgb8_pixel_t(0, 255, 0));
    
    std::cout << "子视图大小: " << sub_view.width() << "x" << sub_view.height() << std::endl;
    
    return 0;
}
```

---

## 参考资源

- [Boost.GIL 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/gil/doc/html/index.html)

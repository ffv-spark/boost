# Boost.Units - 单位库

## 概述

Boost.Units 提供零开销的量纲分析，在编译期检查物理单位的正确性。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 定义物理量
    quantity<length> distance = 100.0 * meters;
    quantity<time_duration> time = 9.8 * seconds;

    // 计算速度
    quantity<velocity> speed = distance / time;

    std::cout << "距离: " << distance << std::endl;
    std::cout << "时间: " << time << std::endl;
    std::cout << "速度: " << speed << std::endl;

    return 0;
}
```

---

## SI 单位系统

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 长度
    quantity<length> l = 5.0 * meters;
    std::cout << "长度: " << l << std::endl;

    // 质量
    quantity<mass> m = 2.0 * kilograms;
    std::cout << "质量: " << m << std::endl;

    // 时间
    quantity<time_duration> t = 10.0 * seconds;
    std::cout << "时间: " << t << std::endl;

    // 力 = 质量 × 加速度
    quantity<force> f = m * (l / (t * t));
    std::cout << "力: " << f << std::endl;

    // 能量 = 力 × 距离
    quantity<energy> e = f * l;
    std::cout << "能量: " << e << std::endl;

    return 0;
}
```

---

## 单位转换

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/systems/cgs.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;

    // SI 单位
    quantity<si::length> meters_val = 1.0 * si::meters;
    std::cout << "SI: " << meters_val << std::endl;

    // 转换为 CGS 单位
    quantity<cgs::length> cm_val = meters_val;
    std::cout << "CGS: " << cm_val << std::endl;

    return 0;
}
```

---

## 自定义单位

```cpp
#include <boost/units/base_units/metric/hour.hpp>
#include <boost/units/systems/si/length.hpp>
#include <boost/units/systems/si/time.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 公里/小时
    typedef divide_typeof_helper<length, time_duration>::type velocity_dim;

    quantity<length> distance = 100.0 * meters;
    quantity<time_duration> time = 10.0 * seconds;

    quantity<velocity_dim> speed = distance / time;

    std::cout << "速度: " << speed << std::endl;
    std::cout << "速度值: " << speed.value() << " m/s" << std::endl;

    return 0;
}
```

---

## 角度

```cpp
#include <boost/units/systems/si/plane_angle.hpp>
#include <boost/units/systems/angle/degrees.hpp>
#include <boost/units/systems/angle/gradians.hpp>
#include <boost/units/io.hpp>
#include <boost/math/constants/constants.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;
    using boost::math::constants::pi;

    // 弧度
    quantity<plane_angle> angle1 = pi<double>() * radians;
    std::cout << "π 弧度 = " << angle1 << std::endl;

    // 度
    quantity<plane_angle> angle2 = 180.0 * degree::degrees;
    std::cout << "180 度 = " << angle2 << std::endl;

    // 转换
    quantity<plane_angle> converted = angle2;
    std::cout << "180度 = " << converted << std::endl;

    return 0;
}
```

---

## 温度

```cpp
#include <boost/units/systems/si/temperature.hpp>
#include <boost/units/systems/temperature/celsius.hpp>
#include <boost/units/systems/temperature/fahrenheit.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;
    using namespace boost::units::celsius;
    using namespace boost::units::fahrenheit;

    // 摄氏度
    quantity<celsius::temperature> temp_c = 25.0 * celsius::temperature();
    std::cout << "摄氏: " << temp_c << std::endl;

    // 转换为开尔文
    quantity<temperature> temp_k(temp_c);
    std::cout << "开尔文: " << temp_k << std::endl;

    return 0;
}
```

---

## 物理常数

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 光速
    const quantity<velocity> c = 299792458.0 * meters_per_second;
    std::cout << "光速: " << c << std::endl;

    // 引力常数
    const quantity<force> g = 9.8 * meters_per_second_squared;
    std::cout << "重力加速度: " << g << std::endl;

    // E = mc²
    quantity<mass> m = 1.0 * kilograms;
    quantity<energy> E = m * c * c;
    std::cout << "能量: " << E << std::endl;

    return 0;
}
```

---

## 量纲分析

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    quantity<length> l = 10.0 * meters;
    quantity<mass> m = 5.0 * kilograms;
    quantity<time_duration> t = 2.0 * seconds;

    // 正确的计算
    quantity<velocity> v = l / t;  // 编译通过
    std::cout << "速度: " << v << std::endl;

    // 以下代码会产生编译错误（量纲不匹配）：
    // quantity<length> wrong = l + m;  // 错误！不能将长度和质量相加
    // quantity<mass> wrong2 = l * t;   // 错误！长度×时间不是质量

    return 0;
}
```

---

## 复合单位

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 密度 = 质量 / 体积
    quantity<mass> m = 1000.0 * kilograms;
    quantity<volume> v = 1.0 * cubic_meters;
    quantity<mass_density> density = m / v;

    std::cout << "密度: " << density << std::endl;

    // 压力 = 力 / 面积
    quantity<force> f = 1000.0 * newtons;
    quantity<area> a = 1.0 * square_meters;
    quantity<pressure> p = f / a;

    std::cout << "压力: " << p << std::endl;

    // 功率 = 能量 / 时间
    quantity<energy> e = 1000.0 * joules;
    quantity<time_duration> t = 1.0 * seconds;
    quantity<power> pow = e / t;

    std::cout << "功率: " << pow << std::endl;

    return 0;
}
```

---

## 无量纲量

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    quantity<length> l1 = 100.0 * meters;
    quantity<length> l2 = 50.0 * meters;

    // 比率（无量纲）
    quantity<dimensionless> ratio = l1 / l2;
    std::cout << "比率: " << ratio << std::endl;
    std::cout << "比率值: " << ratio.value() << std::endl;

    return 0;
}
```

---

## 美国惯用单位

```cpp
#include <boost/units/systems/si.hpp>
#include <boost/units/base_units/us/inch.hpp>
#include <boost/units/base_units/us/pound.hpp>
#include <boost/units/io.hpp>
#include <iostream>

int main() {
    using namespace boost::units;
    using namespace boost::units::si;

    // 英寸
    quantity<si::length> inches_val = 12.0 * us::inch_base_unit::unit_type();

    // 转换为米
    std::cout << "12 英寸 = " << inches_val << std::endl;

    return 0;
}
```

---

## 参考资源

- [Boost.Units 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_units.html)
- [单位系统](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_units/Units.html)

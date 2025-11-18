# Boost.Odeint - 常微分方程求解器

## 概述

Boost.Odeint 提供数值求解常微分方程（ODE）的工具，支持多种积分方法。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/numeric/odeint.hpp>
#include <iostream>
#include <vector>

namespace odeint = boost::numeric::odeint;

// 定义ODE: dx/dt = x
void harmonic_oscillator(const std::vector<double>& x,
                        std::vector<double>& dxdt,
                        double t) {
    dxdt[0] = x[0];  // 简单增长模型
}

int main() {
    std::vector<double> x(1, 1.0);  // 初始条件 x(0) = 1.0
    
    // 使用Runge-Kutta 4方法
    odeint::runge_kutta4<std::vector<double>> stepper;
    
    double t = 0.0;
    double dt = 0.1;
    
    for (int i = 0; i < 10; ++i) {
        std::cout << "t = " << t << ", x = " << x[0] << std::endl;
        stepper.do_step(harmonic_oscillator, x, t, dt);
        t += dt;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Odeint 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/numeric/odeint/doc/html/index.html)

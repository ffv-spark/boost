# Boost.Math - 数学库

## 概述

Boost.Math 提供丰富的数学函数和工具，包括特殊函数、统计分布、数值算法等。

**类型**: 仅头文件库（大部分功能）

---

## 快速开始

```cpp
#include <boost/math/special_functions.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::math;

    // 阶乘
    std::cout << "5! = " << factorial<double>(5) << std::endl;

    // 组合数 C(10, 3)
    std::cout << "C(10,3) = " << binomial_coefficient<double>(10, 3) << std::endl;

    // 最大公约数
    std::cout << "gcd(48, 18) = " << gcd(48, 18) << std::endl;

    // 最小公倍数
    std::cout << "lcm(12, 18) = " << lcm(12, 18) << std::endl;

    return 0;
}
```

---

## 特殊函数

```cpp
#include <boost/math/special_functions.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::math;

    std::cout << std::setprecision(10);

    // 伽马函数
    std::cout << "Γ(5) = " << tgamma(5.0) << std::endl;

    // 贝塔函数
    std::cout << "B(3, 4) = " << beta(3.0, 4.0) << std::endl;

    // 误差函数
    std::cout << "erf(1) = " << erf(1.0) << std::endl;

    // 贝塞尔函数
    std::cout << "J₀(2) = " << cyl_bessel_j(0, 2.0) << std::endl;

    // 勒让德多项式
    std::cout << "P₃(0.5) = " << legendre_p(3, 0.5) << std::endl;

    return 0;
}
```

---

## 统计分布

```cpp
#include <boost/math/distributions.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::math;

    std::cout << std::setprecision(6);

    // 正态分布
    normal_distribution<> norm(0.0, 1.0);  // 均值0，标准差1

    std::cout << "正态分布 N(0,1):\n";
    std::cout << "  PDF(0) = " << pdf(norm, 0.0) << std::endl;
    std::cout << "  CDF(1.96) = " << cdf(norm, 1.96) << std::endl;
    std::cout << "  95%分位数 = " << quantile(norm, 0.95) << std::endl;

    // 二项分布
    binomial_distribution<> binom(10, 0.5);  // n=10, p=0.5

    std::cout << "\n二项分布 B(10, 0.5):\n";
    std::cout << "  P(X=5) = " << pdf(binom, 5) << std::endl;
    std::cout << "  P(X≤5) = " << cdf(binom, 5) << std::endl;

    // 泊松分布
    poisson_distribution<> pois(3.0);  // λ=3

    std::cout << "\n泊松分布 P(3):\n";
    std::cout << "  P(X=2) = " << pdf(pois, 2) << std::endl;
    std::cout << "  P(X≤4) = " << cdf(pois, 4) << std::endl;

    return 0;
}
```

---

## 数值微分

```cpp
#include <boost/math/differentiation/finite_difference.hpp>
#include <iostream>
#include <cmath>
#include <iomanip>

double f(double x) {
    return x * x * x - 2 * x * x + x - 1;
}

int main() {
    using namespace boost::math::differentiation;

    double x = 2.0;

    // 一阶导数
    double df = finite_difference_derivative(f, x);
    std::cout << std::setprecision(10);
    std::cout << "f'(2) ≈ " << df << std::endl;

    // 解析解：f'(x) = 3x² - 4x + 1, f'(2) = 5
    std::cout << "理论值: " << (3 * x * x - 4 * x + 1) << std::endl;

    return 0;
}
```

---

## 数值积分

```cpp
#include <boost/math/quadrature/trapezoidal.hpp>
#include <boost/math/quadrature/gauss.hpp>
#include <iostream>
#include <cmath>
#include <iomanip>

int main() {
    using namespace boost::math::quadrature;

    // 被积函数
    auto f = [](double x) { return std::exp(-x * x); };

    // 梯形法则
    double result_trap = trapezoidal(f, 0.0, 1.0, 1e-6);
    std::cout << std::setprecision(10);
    std::cout << "梯形积分: " << result_trap << std::endl;

    // 高斯积分
    gauss<double, 15> integrator;
    double result_gauss = integrator.integrate(f, 0.0, 1.0);
    std::cout << "高斯积分: " << result_gauss << std::endl;

    return 0;
}
```

---

## 根查找

```cpp
#include <boost/math/tools/roots.hpp>
#include <iostream>
#include <cmath>

// 求解 x² - 2 = 0
struct equation {
    double operator()(double x) const {
        return x * x - 2.0;
    }
};

int main() {
    using namespace boost::math::tools;

    equation eq;

    // 二分法
    double min = 0.0, max = 2.0;
    std::pair<double, double> result = bisect(
        eq,
        min, max,
        [](double a, double b) { return std::abs(a - b) < 1e-10; }
    );

    double root = (result.first + result.second) / 2.0;
    std::cout << "√2 ≈ " << root << std::endl;
    std::cout << "误差: " << std::abs(root - std::sqrt(2.0)) << std::endl;

    return 0;
}
```

---

## 常数

```cpp
#include <boost/math/constants/constants.hpp>
#include <iostream>
#include <iomanip>

int main() {
    using namespace boost::math::constants;

    std::cout << std::setprecision(15);

    std::cout << "π = " << pi<double>() << std::endl;
    std::cout << "e = " << e<double>() << std::endl;
    std::cout << "√2 = " << root_two<double>() << std::endl;
    std::cout << "ln(2) = " << ln_two<double>() << std::endl;
    std::cout << "1/√(2π) = " << one_div_root_two_pi<double>() << std::endl;

    // 黄金比例
    std::cout << "φ = " << phi<double>() << std::endl;

    return 0;
}
```

---

## 插值

```cpp
#include <boost/math/interpolators/cubic_b_spline.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::math::interpolators;

    // 数据点
    std::vector<double> y = {0.0, 1.0, 4.0, 9.0, 16.0};  // y = x²
    double x_start = 0.0;
    double step = 1.0;

    // 创建三次B样条插值
    cubic_b_spline<double> spline(y.begin(), y.end(), x_start, step);

    // 插值
    std::cout << "x = 0.5, 插值 y = " << spline(0.5) << std::endl;
    std::cout << "x = 1.5, 插值 y = " << spline(1.5) << std::endl;
    std::cout << "x = 2.5, 插值 y = " << spline(2.5) << std::endl;

    // 实际值
    std::cout << "\n实际值:\n";
    std::cout << "0.5² = " << 0.5 * 0.5 << std::endl;
    std::cout << "1.5² = " << 1.5 * 1.5 << std::endl;
    std::cout << "2.5² = " << 2.5 * 2.5 << std::endl;

    return 0;
}
```

---

## 四元数

```cpp
#include <boost/math/quaternion.hpp>
#include <iostream>

int main() {
    using namespace boost::math;

    // 创建四元数
    quaternion<double> q1(1, 2, 3, 4);  // 1 + 2i + 3j + 4k
    quaternion<double> q2(5, 6, 7, 8);

    std::cout << "q1 = " << q1 << std::endl;
    std::cout << "q2 = " << q2 << std::endl;

    // 四元数运算
    std::cout << "q1 + q2 = " << (q1 + q2) << std::endl;
    std::cout << "q1 * q2 = " << (q1 * q2) << std::endl;

    // 共轭
    std::cout << "conj(q1) = " << conj(q1) << std::endl;

    // 模
    std::cout << "|q1| = " << abs(q1) << std::endl;

    return 0;
}
```

---

## 八元数

```cpp
#include <boost/math/octonion.hpp>
#include <iostream>

int main() {
    using namespace boost::math;

    // 创建八元数
    octonion<double> o1(1, 2, 3, 4, 5, 6, 7, 8);
    octonion<double> o2(8, 7, 6, 5, 4, 3, 2, 1);

    std::cout << "o1 = " << o1 << std::endl;
    std::cout << "o2 = " << o2 << std::endl;

    // 八元数运算
    std::cout << "o1 + o2 = " << (o1 + o2) << std::endl;
    std::cout << "o1 * o2 = " << (o1 * o2) << std::endl;

    // 模
    std::cout << "|o1| = " << abs(o1) << std::endl;

    return 0;
}
```

---

## 统计函数

```cpp
#include <boost/math/statistics/univariate_statistics.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::math::statistics;

    std::vector<double> data = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0};

    // 均值
    double mean_val = mean(data);
    std::cout << "均值: " << mean_val << std::endl;

    // 方差
    double var_val = variance(data);
    std::cout << "方差: " << var_val << std::endl;

    // 标准差
    double std_val = sample_variance(data);
    std::cout << "标准差: " << std::sqrt(std_val) << std::endl;

    // 中位数
    double median_val = median(data.begin(), data.end());
    std::cout << "中位数: " << median_val << std::endl;

    return 0;
}
```

---

## 假设检验

```cpp
#include <boost/math/distributions/students_t.hpp>
#include <iostream>
#include <vector>
#include <cmath>

int main() {
    using namespace boost::math;

    // 样本数据
    std::vector<double> sample = {23.5, 24.1, 23.8, 24.3, 23.9};

    // 计算样本均值和标准差
    double sum = 0.0;
    for (double x : sample) sum += x;
    double mean = sum / sample.size();

    double sq_sum = 0.0;
    for (double x : sample) {
        sq_sum += (x - mean) * (x - mean);
    }
    double std_dev = std::sqrt(sq_sum / (sample.size() - 1));

    // t检验：检验均值是否为24
    double mu0 = 24.0;
    double t_stat = (mean - mu0) / (std_dev / std::sqrt(sample.size()));

    students_t dist(sample.size() - 1);
    double p_value = 2.0 * cdf(complement(dist, std::abs(t_stat)));

    std::cout << "样本均值: " << mean << std::endl;
    std::cout << "t统计量: " << t_stat << std::endl;
    std::cout << "p值: " << p_value << std::endl;

    if (p_value < 0.05) {
        std::cout << "拒绝原假设（显著性水平0.05）" << std::endl;
    } else {
        std::cout << "不能拒绝原假设" << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Math 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/math/doc/html/index.html)
- [特殊函数参考](https://www.boost.org/doc/libs/1_90_0/libs/math/doc/html/special.html)
- [统计分布](https://www.boost.org/doc/libs/1_90_0/libs/math/doc/html/dist.html)

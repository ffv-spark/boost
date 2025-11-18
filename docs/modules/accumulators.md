# Boost.Accumulators - 累加器库

## 概述

Boost.Accumulators 提供增量统计计算框架，可以在数据流中高效计算各种统计量。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>

int main() {
    using namespace boost::accumulators;

    // 定义累加器，计算均值和方差
    accumulator_set<double, features<tag::mean, tag::variance>> acc;

    // 添加数据
    acc(1.0);
    acc(2.0);
    acc(3.0);
    acc(4.0);
    acc(5.0);

    std::cout << "均值: " << mean(acc) << std::endl;
    std::cout << "方差: " << variance(acc) << std::endl;

    return 0;
}
```

---

## 基本统计量

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::accumulators;

    // 定义累加器
    accumulator_set<double, features<
        tag::count,
        tag::sum,
        tag::mean,
        tag::variance,
        tag::min,
        tag::max
    >> acc;

    // 数据
    std::vector<double> data = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0};

    // 添加数据
    for (double x : data) {
        acc(x);
    }

    // 输出统计量
    std::cout << "计数: " << count(acc) << std::endl;
    std::cout << "总和: " << sum(acc) << std::endl;
    std::cout << "均值: " << mean(acc) << std::endl;
    std::cout << "方差: " << variance(acc) << std::endl;
    std::cout << "标准差: " << std::sqrt(variance(acc)) << std::endl;
    std::cout << "最小值: " << (min)(acc) << std::endl;
    std::cout << "最大值: " << (max)(acc) << std::endl;

    return 0;
}
```

---

## 分位数

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>
#include <vector>

int main() {
    using namespace boost::accumulators;

    // 定义累加器，计算中位数和四分位数
    accumulator_set<double, features<
        tag::median,
        tag::tail_quantile<left>,
        tag::tail_quantile<right>
    >> acc(tag::tail<left>::cache_size = 100,
           tag::tail<right>::cache_size = 100);

    // 添加数据
    for (int i = 1; i <= 100; ++i) {
        acc(i);
    }

    std::cout << "中位数: " << median(acc) << std::endl;
    std::cout << "25%分位数: " << quantile(acc, quantile_probability = 0.25) << std::endl;
    std::cout << "75%分位数: " << quantile(acc, quantile_probability = 0.75) << std::endl;

    return 0;
}
```

---

## 移动窗口统计

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>

int main() {
    using namespace boost::accumulators;

    // 5个数据的移动窗口
    accumulator_set<double, features<
        tag::rolling_mean,
        tag::rolling_variance
    >> acc(tag::rolling_window::window_size = 5);

    // 添加数据
    for (int i = 1; i <= 10; ++i) {
        acc(i);
        std::cout << "第" << i << "个数据后:" << std::endl;
        std::cout << "  移动均值: " << rolling_mean(acc) << std::endl;
        if (i >= 2) {
            std::cout << "  移动方差: " << rolling_variance(acc) << std::endl;
        }
        std::cout << std::endl;
    }

    return 0;
}
```

---

## 加权统计

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>

int main() {
    using namespace boost::accumulators;

    // 加权累加器
    accumulator_set<double, features<
        tag::weighted_mean,
        tag::weighted_variance
    >, double> acc;  // 第三个参数指定权重类型

    // 添加加权数据
    acc(1.0, weight = 1.0);
    acc(2.0, weight = 2.0);
    acc(3.0, weight = 3.0);
    acc(4.0, weight = 4.0);
    acc(5.0, weight = 5.0);

    std::cout << "加权均值: " << weighted_mean(acc) << std::endl;
    std::cout << "加权方差: " << weighted_variance(acc) << std::endl;

    return 0;
}
```

---

## 峰度和偏度

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>
#include <vector>
#include <random>

int main() {
    using namespace boost::accumulators;

    accumulator_set<double, features<
        tag::mean,
        tag::variance,
        tag::skewness,
        tag::kurtosis
    >> acc;

    // 生成正态分布数据
    std::random_device rd;
    std::mt19937 gen(rd());
    std::normal_distribution<> d(0, 1);

    for (int i = 0; i < 10000; ++i) {
        acc(d(gen));
    }

    std::cout << "均值: " << mean(acc) << std::endl;
    std::cout << "方差: " << variance(acc) << std::endl;
    std::cout << "偏度: " << skewness(acc) << std::endl;
    std::cout << "峰度: " << kurtosis(acc) << std::endl;

    return 0;
}
```

---

## 协方差

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>

int main() {
    using namespace boost::accumulators;

    // 计算两个变量的协方差
    accumulator_set<double, features<
        tag::covariance<double, tag::covariate1>
    >> acc;

    // 添加成对数据
    acc(1.0, covariate1 = 2.0);
    acc(2.0, covariate1 = 4.0);
    acc(3.0, covariate1 = 6.0);
    acc(4.0, covariate1 = 8.0);
    acc(5.0, covariate1 = 10.0);

    std::cout << "协方差: " << covariance(acc) << std::endl;

    return 0;
}
```

---

## 时间序列

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>
#include <cmath>

int main() {
    using namespace boost::accumulators;

    // 指数加权移动平均
    accumulator_set<double, features<
        tag::ewma
    >> acc(tag::ewma::alpha = 0.3);  // 平滑因子

    // 添加数据
    for (int i = 1; i <= 10; ++i) {
        double value = std::sin(i * 0.5) * 10 + 50;
        acc(value);
        std::cout << "第" << i << "个数据: " << value
                  << ", EWMA: " << ewma(acc) << std::endl;
    }

    return 0;
}
```

---

## 自定义累加器

```cpp
#include <boost/accumulators/framework/accumulator_base.hpp>
#include <boost/accumulators/framework/extractor.hpp>
#include <boost/accumulators/framework/parameters/sample.hpp>
#include <iostream>

namespace boost { namespace accumulators {

// 自定义累加器：计算样本乘积
namespace impl {
    template<typename Sample>
    struct product_impl : accumulator_base {
        typedef Sample result_type;

        product_impl(dont_care) : product(1) {}

        template<typename Args>
        void operator()(Args const& args) {
            product *= args[sample];
        }

        result_type result(dont_care) const {
            return product;
        }

        Sample product;
    };
}

namespace tag {
    struct product : depends_on<> {
        typedef accumulators::impl::product_impl<mpl::_1> impl;
    };
}

namespace extract {
    extractor<tag::product> const product = {};
}

using extract::product;

}} // namespace boost::accumulators

int main() {
    using namespace boost::accumulators;

    accumulator_set<double, features<tag::product>> acc;

    acc(2.0);
    acc(3.0);
    acc(4.0);

    std::cout << "乘积: " << product(acc) << std::endl;

    return 0;
}
```

---

## 数据流处理

```cpp
#include <boost/accumulators/accumulators.hpp>
#include <boost/accumulators/statistics.hpp>
#include <iostream>
#include <random>

int main() {
    using namespace boost::accumulators;

    accumulator_set<double, features<
        tag::count,
        tag::mean,
        tag::variance,
        tag::min,
        tag::max
    >> acc;

    // 模拟实时数据流
    std::random_device rd;
    std::mt19937 gen(rd());
    std::normal_distribution<> d(100, 15);

    std::cout << "实时数据流统计:\n";
    for (int i = 1; i <= 20; ++i) {
        double value = d(gen);
        acc(value);

        if (i % 5 == 0) {
            std::cout << "\n前" << i << "个数据:" << std::endl;
            std::cout << "  均值: " << mean(acc) << std::endl;
            std::cout << "  标准差: " << std::sqrt(variance(acc)) << std::endl;
            std::cout << "  范围: [" << (min)(acc) << ", " << (max)(acc) << "]" << std::endl;
        }
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Accumulators 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/accumulators.html)
- [统计特性参考](https://www.boost.org/doc/libs/1_90_0/doc/html/accumulators/user_s_guide.html)

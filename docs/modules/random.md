# Boost.Random - 随机数库

## 概述

Boost.Random 提供高质量的随机数生成器和分布函数。

**类型**: 仅头文件库

**注意**: C++11 引入了 `<random>`，但 Boost.Random 提供了更多选项

---

## 快速开始

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    // 创建随机数生成器
    boost::random::mt19937 rng;

    // 创建分布
    boost::random::uniform_int_distribution<> dist(1, 6);

    // 生成随机数
    std::cout << "掷骰子5次:\n";
    for (int i = 0; i < 5; ++i) {
        std::cout << "  " << dist(rng) << std::endl;
    }

    return 0;
}
```

---

## 基本随机数生成

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    // 使用时间作为种子
    boost::random::mt19937 rng(static_cast<unsigned>(std::time(nullptr)));

    std::cout << "10个随机数:\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << "  " << rng() << std::endl;
    }

    return 0;
}
```

---

## 均匀分布

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    boost::random::mt19937 rng;

    // 整数均匀分布 [1, 100]
    boost::random::uniform_int_distribution<> int_dist(1, 100);

    std::cout << "10个随机整数 [1, 100]:\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << "  " << int_dist(rng) << std::endl;
    }

    // 实数均匀分布 [0.0, 1.0)
    boost::random::uniform_real_distribution<> real_dist(0.0, 1.0);

    std::cout << "\n10个随机实数 [0.0, 1.0):\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << "  " << real_dist(rng) << std::endl;
    }

    return 0;
}
```

---

## 正态分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>

int main() {
    boost::random::mt19937 rng;

    // 正态分布：均值100，标准差15
    boost::random::normal_distribution<> dist(100.0, 15.0);

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "正态分布样本（均值100，标准差15）:\n";
    for (int i = 0; i < 20; ++i) {
        std::cout << "  " << dist(rng) << std::endl;
    }

    return 0;
}
```

---

## 随机数生成器对比

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <chrono>

template<typename RNG>
void benchmark(const char* name) {
    RNG rng;
    const int iterations = 10000000;

    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        rng();
    }
    auto end = std::chrono::high_resolution_clock::now();

    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << name << ": " << duration.count() << " ms" << std::endl;
}

int main() {
    std::cout << "生成1000万个随机数:\n";

    benchmark<boost::random::mt19937>("Mersenne Twister");
    benchmark<boost::random::rand48>("rand48");
    benchmark<boost::random::minstd_rand>("minstd_rand");

    return 0;
}
```

---

## 泊松分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <map>

int main() {
    boost::random::mt19937 rng;

    // 泊松分布：平均值为5
    boost::random::poisson_distribution<> dist(5.0);

    // 统计分布
    std::map<int, int> histogram;
    for (int i = 0; i < 10000; ++i) {
        ++histogram[dist(rng)];
    }

    std::cout << "泊松分布直方图（均值=5）:\n";
    for (const auto& p : histogram) {
        std::cout << p.first << ": " << std::string(p.second / 100, '*') << std::endl;
    }

    return 0;
}
```

---

## 指数分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>

int main() {
    boost::random::mt19937 rng;

    // 指数分布：lambda=2.0
    boost::random::exponential_distribution<> dist(2.0);

    std::cout << std::fixed << std::setprecision(4);
    std::cout << "指数分布样本（lambda=2.0）:\n";
    for (int i = 0; i < 20; ++i) {
        std::cout << "  " << dist(rng) << std::endl;
    }

    return 0;
}
```

---

## 二项分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <map>

int main() {
    boost::random::mt19937 rng;

    // 二项分布：10次试验，成功概率0.5
    boost::random::binomial_distribution<> dist(10, 0.5);

    // 统计分布
    std::map<int, int> histogram;
    for (int i = 0; i < 10000; ++i) {
        ++histogram[dist(rng)];
    }

    std::cout << "二项分布直方图（n=10, p=0.5）:\n";
    for (const auto& p : histogram) {
        std::cout << p.first << ": " << std::string(p.second / 50, '*') << std::endl;
    }

    return 0;
}
```

---

## 随机采样

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    boost::random::mt19937 rng;

    std::vector<int> data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 打乱顺序
    boost::random::shuffle(data.begin(), data.end(), rng);

    std::cout << "打乱后: ";
    for (int x : data) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 随机选择一个元素
    boost::random::uniform_int_distribution<> dist(0, data.size() - 1);
    int index = dist(rng);
    std::cout << "随机选择: " << data[index] << std::endl;

    return 0;
}
```

---

## 蒙特卡洛模拟

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>

// 估算 π
double estimate_pi(int samples) {
    boost::random::mt19937 rng;
    boost::random::uniform_real_distribution<> dist(0.0, 1.0);

    int inside_circle = 0;
    for (int i = 0; i < samples; ++i) {
        double x = dist(rng);
        double y = dist(rng);
        if (x * x + y * y <= 1.0) {
            ++inside_circle;
        }
    }

    return 4.0 * inside_circle / samples;
}

int main() {
    std::cout << std::fixed << std::setprecision(6);

    for (int samples : {1000, 10000, 100000, 1000000}) {
        double pi = estimate_pi(samples);
        std::cout << "样本数 " << samples << ": π ≈ " << pi << std::endl;
    }

    return 0;
}
```

---

## 随机密码生成

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <string>

std::string generate_password(int length) {
    boost::random::mt19937 rng(static_cast<unsigned>(std::time(nullptr)));

    std::string charset = 
        "0123456789"
        "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
        "abcdefghijklmnopqrstuvwxyz"
        "!@#$%^&*";

    boost::random::uniform_int_distribution<> dist(0, charset.length() - 1);

    std::string password;
    for (int i = 0; i < length; ++i) {
        password += charset[dist(rng)];
    }

    return password;
}

int main() {
    std::cout << "生成5个随机密码:\n";
    for (int i = 0; i < 5; ++i) {
        std::cout << "  " << generate_password(16) << std::endl;
    }

    return 0;
}
```

---

## 随机点生成

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <vector>
#include <iomanip>

struct Point {
    double x, y;
};

int main() {
    boost::random::mt19937 rng;
    boost::random::uniform_real_distribution<> dist(-10.0, 10.0);

    std::vector<Point> points;

    // 生成20个随机点
    for (int i = 0; i < 20; ++i) {
        points.push_back({dist(rng), dist(rng)});
    }

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "随机点:\n";
    for (const auto& p : points) {
        std::cout << "  (" << p.x << ", " << p.y << ")" << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Random 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_random.html)
- [C++ `<random>` 参考](https://en.cppreference.com/w/cpp/numeric/random)

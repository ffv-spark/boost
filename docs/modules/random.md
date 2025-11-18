# Boost.Random - 随机数生成库

## 概述

Boost.Random 提供了丰富的随机数生成器和分布，用于生成高质量的随机数。

**类型**: 仅头文件库

**注意**: C++11 已引入 `<random>`，功能基本一致

**主要组件**:
- 随机数引擎（Generators）
- 分布（Distributions）
- 非确定性随机数生成器

---

## 快速开始

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    // 随机数引擎
    boost::random::mt19937 rng;

    // 均匀分布 [1, 6]
    boost::random::uniform_int_distribution<> dice(1, 6);

    // 生成10个骰子点数
    for (int i = 0; i < 10; ++i) {
        std::cout << dice(rng) << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -o example
```

---

## 随机数引擎

### 常用引擎

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    // 1. Mersenne Twister（推荐，高质量）
    boost::random::mt19937 mt_rng;

    // 2. 线性同余生成器（快速，质量较低）
    boost::random::minstd_rand lcg_rng;

    // 3. Lagged Fibonacci
    boost::random::lagged_fibonacci607 fib_rng;

    // 4. 使用种子
    boost::random::mt19937 seeded_rng(12345);

    // 5. 使用当前时间作为种子
    boost::random::mt19937 time_rng(static_cast<unsigned>(time(nullptr)));

    // 生成随机数
    std::cout << "MT19937: " << mt_rng() << std::endl;
    std::cout << "LCG: " << lcg_rng() << std::endl;
    std::cout << "带种子: " << seeded_rng() << std::endl;

    return 0;
}
```

### 引擎对比

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <chrono>

template<typename Engine>
void benchmark(const std::string& name) {
    Engine rng;

    auto start = std::chrono::high_resolution_clock::now();

    long long sum = 0;
    for (int i = 0; i < 1000000; ++i) {
        sum += rng();
    }

    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << name << ": " << duration.count() << " ms" << std::endl;
}

int main() {
    benchmark<boost::random::mt19937>("MT19937");
    benchmark<boost::random::minstd_rand>("MINSTD");
    benchmark<boost::random::lagged_fibonacci607>("Lagged Fibonacci");

    return 0;
}
```

---

## 均匀分布

### 整数均匀分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <map>

int main() {
    boost::random::mt19937 rng;

    // 1. 骰子 [1, 6]
    boost::random::uniform_int_distribution<> dice(1, 6);

    std::cout << "掷骰子10次: ";
    for (int i = 0; i < 10; ++i) {
        std::cout << dice(rng) << " ";
    }
    std::cout << "\n" << std::endl;

    // 2. 统计分布
    std::map<int, int> histogram;
    for (int i = 0; i < 10000; ++i) {
        ++histogram[dice(rng)];
    }

    std::cout << "分布统计（10000次）:\n";
    for (const auto& [value, count] : histogram) {
        std::cout << value << ": " << std::string(count / 100, '*') << " " << count << std::endl;
    }

    return 0;
}
```

### 浮点数均匀分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>

int main() {
    boost::random::mt19937 rng;

    // [0.0, 1.0) 均匀分布
    boost::random::uniform_real_distribution<> unit(0.0, 1.0);

    // [-1.0, 1.0) 均匀分布
    boost::random::uniform_real_distribution<> symmetric(-1.0, 1.0);

    std::cout << std::fixed << std::setprecision(6);

    std::cout << "单位分布:\n";
    for (int i = 0; i < 5; ++i) {
        std::cout << unit(rng) << std::endl;
    }

    std::cout << "\n对称分布:\n";
    for (int i = 0; i < 5; ++i) {
        std::cout << symmetric(rng) << std::endl;
    }

    return 0;
}
```

---

## 常见分布

### 正态分布（高斯分布）

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>
#include <map>
#include <cmath>

int main() {
    boost::random::mt19937 rng;

    // 均值0，标准差1的正态分布
    boost::random::normal_distribution<> normal(0.0, 1.0);

    // 自定义均值和标准差
    boost::random::normal_distribution<> custom_normal(100.0, 15.0);  // IQ分数

    std::cout << std::fixed << std::setprecision(3);

    // 生成样本
    std::cout << "标准正态分布:\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << normal(rng) << " ";
    }
    std::cout << "\n" << std::endl;

    // 统计直方图
    std::map<int, int> histogram;
    for (int i = 0; i < 100000; ++i) {
        double value = custom_normal(rng);
        int bucket = static_cast<int>(std::round(value / 10)) * 10;
        ++histogram[bucket];
    }

    std::cout << "IQ 分数分布:\n";
    for (const auto& [bucket, count] : histogram) {
        if (count > 100) {
            std::cout << bucket << ": "
                      << std::string(count / 1000, '*')
                      << " " << count << std::endl;
        }
    }

    return 0;
}
```

### 指数分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <iomanip>

int main() {
    boost::random::mt19937 rng;

    // 指数分布（λ = 1.0）
    boost::random::exponential_distribution<> exp_dist(1.0);

    // 模拟事件到达时间间隔（平均间隔1秒）
    std::cout << std::fixed << std::setprecision(3);

    std::cout << "事件间隔时间（秒）:\n";
    double total_time = 0.0;
    for (int i = 0; i < 10; ++i) {
        double interval = exp_dist(rng);
        total_time += interval;
        std::cout << "事件 " << (i + 1)
                  << " 在时刻 " << total_time
                  << " 秒（间隔 " << interval << " 秒）" << std::endl;
    }

    return 0;
}
```

### 泊松分布

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <map>

int main() {
    boost::random::mt19937 rng;

    // 泊松分布（λ = 4.0，平均每小时4个事件）
    boost::random::poisson_distribution<> poisson(4.0);

    std::cout << "每小时事件数量:\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << "小时 " << (i + 1) << ": " << poisson(rng) << " 个事件" << std::endl;
    }

    // 统计
    std::map<int, int> histogram;
    for (int i = 0; i < 10000; ++i) {
        ++histogram[poisson(rng)];
    }

    std::cout << "\n分布统计:\n";
    for (const auto& [events, count] : histogram) {
        std::cout << events << " 个事件: "
                  << std::string(count / 100, '*')
                  << " " << count << std::endl;
    }

    return 0;
}
```

### 伯努利分布

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    boost::random::mt19937 rng;

    // 抛硬币（50%概率）
    boost::random::bernoulli_distribution<> coin(0.5);

    // 有偏硬币（30%概率正面）
    boost::random::bernoulli_distribution<> biased_coin(0.3);

    std::cout << "抛公平硬币10次:\n";
    int heads = 0;
    for (int i = 0; i < 10; ++i) {
        bool result = coin(rng);
        std::cout << (result ? "H" : "T") << " ";
        if (result) ++heads;
    }
    std::cout << "\n正面: " << heads << " 次\n" << std::endl;

    std::cout << "抛有偏硬币10次:\n";
    heads = 0;
    for (int i = 0; i < 10; ++i) {
        bool result = biased_coin(rng);
        std::cout << (result ? "H" : "T") << " ";
        if (result) ++heads;
    }
    std::cout << "\n正面: " << heads << " 次" << std::endl;

    return 0;
}
```

---

## 实用示例

### 随机采样

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

template<typename T>
std::vector<T> random_sample(const std::vector<T>& population, size_t sample_size) {
    boost::random::mt19937 rng(static_cast<unsigned>(time(nullptr)));

    std::vector<T> result;
    std::vector<size_t> indices(population.size());
    std::iota(indices.begin(), indices.end(), 0);

    // Fisher-Yates shuffle
    for (size_t i = 0; i < sample_size && i < population.size(); ++i) {
        boost::random::uniform_int_distribution<size_t> dist(i, population.size() - 1);
        size_t j = dist(rng);
        std::swap(indices[i], indices[j]);
        result.push_back(population[indices[i]]);
    }

    return result;
}

int main() {
    std::vector<int> population = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    auto sample = random_sample(population, 5);

    std::cout << "随机采样5个元素: ";
    for (int x : sample) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### 蒙特卡洛模拟

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <cmath>

// 使用蒙特卡洛方法估算 π
double estimate_pi(int num_samples) {
    boost::random::mt19937 rng;
    boost::random::uniform_real_distribution<> dist(0.0, 1.0);

    int inside_circle = 0;

    for (int i = 0; i < num_samples; ++i) {
        double x = dist(rng);
        double y = dist(rng);

        if (x * x + y * y <= 1.0) {
            ++inside_circle;
        }
    }

    return 4.0 * inside_circle / num_samples;
}

int main() {
    std::vector<int> sample_sizes = {1000, 10000, 100000, 1000000};

    for (int size : sample_sizes) {
        double pi_estimate = estimate_pi(size);
        double error = std::abs(pi_estimate - M_PI);

        std::cout << "样本数 " << size << ": "
                  << "π ≈ " << pi_estimate
                  << " (误差: " << error << ")" << std::endl;
    }

    return 0;
}
```

### 模拟退火

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <cmath>
#include <vector>

// 目标函数（最小化）
double objective(const std::vector<double>& x) {
    double sum = 0.0;
    for (double val : x) {
        sum += val * val;
    }
    return sum;
}

std::vector<double> simulated_annealing(int dimensions, int iterations) {
    boost::random::mt19937 rng;
    boost::random::uniform_real_distribution<> init_dist(-10.0, 10.0);
    boost::random::normal_distribution<> move_dist(0.0, 1.0);
    boost::random::uniform_real_distribution<> accept_dist(0.0, 1.0);

    // 初始解
    std::vector<double> current(dimensions);
    for (double& x : current) {
        x = init_dist(rng);
    }

    double current_cost = objective(current);
    std::vector<double> best = current;
    double best_cost = current_cost;

    // 退火过程
    for (int iter = 0; iter < iterations; ++iter) {
        double temperature = 100.0 * (1.0 - static_cast<double>(iter) / iterations);

        // 生成邻近解
        std::vector<double> neighbor = current;
        for (double& x : neighbor) {
            x += move_dist(rng) * temperature / 100.0;
        }

        double neighbor_cost = objective(neighbor);
        double delta = neighbor_cost - current_cost;

        // 接受准则
        if (delta < 0 || accept_dist(rng) < std::exp(-delta / temperature)) {
            current = neighbor;
            current_cost = neighbor_cost;

            if (current_cost < best_cost) {
                best = current;
                best_cost = current_cost;
            }
        }
    }

    std::cout << "最优值: " << best_cost << std::endl;
    return best;
}

int main() {
    auto solution = simulated_annealing(10, 10000);

    std::cout << "最优解: [";
    for (size_t i = 0; i < solution.size(); ++i) {
        std::cout << solution[i];
        if (i < solution.size() - 1) std::cout << ", ";
    }
    std::cout << "]" << std::endl;

    return 0;
}
```

### 随机密码生成

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <string>

std::string generate_password(int length, bool use_special = true) {
    boost::random::mt19937 rng(static_cast<unsigned>(time(nullptr)));

    std::string charset = "0123456789"
                          "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
                          "abcdefghijklmnopqrstuvwxyz";

    if (use_special) {
        charset += "!@#$%^&*()_+-=[]{}|;:,.<>?";
    }

    boost::random::uniform_int_distribution<> dist(0, charset.size() - 1);

    std::string password;
    for (int i = 0; i < length; ++i) {
        password += charset[dist(rng)];
    }

    return password;
}

int main() {
    std::cout << "随机密码（8位）: " << generate_password(8) << std::endl;
    std::cout << "随机密码（16位）: " << generate_password(16) << std::endl;
    std::cout << "随机密码（12位，无特殊字符）: " << generate_password(12, false) << std::endl;

    return 0;
}
```

### 随机洗牌

```cpp
#include <boost/random.hpp>
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

template<typename T>
void random_shuffle(std::vector<T>& vec) {
    boost::random::mt19937 rng(static_cast<unsigned>(time(nullptr)));

    for (size_t i = vec.size() - 1; i > 0; --i) {
        boost::random::uniform_int_distribution<size_t> dist(0, i);
        size_t j = dist(rng);
        std::swap(vec[i], vec[j]);
    }
}

int main() {
    // 洗牌扑克牌
    std::vector<std::string> deck;
    std::vector<std::string> suits = {"♠", "♥", "♦", "♣"};
    std::vector<std::string> ranks = {"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"};

    for (const auto& suit : suits) {
        for (const auto& rank : ranks) {
            deck.push_back(rank + suit);
        }
    }

    std::cout << "原始牌组（前10张）:\n";
    for (size_t i = 0; i < 10; ++i) {
        std::cout << deck[i] << " ";
    }
    std::cout << "\n" << std::endl;

    random_shuffle(deck);

    std::cout << "洗牌后（前10张）:\n";
    for (size_t i = 0; i < 10; ++i) {
        std::cout << deck[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## variate_generator（兼容旧代码）

```cpp
#include <boost/random.hpp>
#include <iostream>

int main() {
    boost::random::mt19937 rng;
    boost::random::uniform_int_distribution<> dist(1, 100);

    // 使用 variate_generator 组合引擎和分布
    boost::variate_generator<boost::random::mt19937&, boost::random::uniform_int_distribution<>>
        generator(rng, dist);

    std::cout << "使用 variate_generator:\n";
    for (int i = 0; i < 10; ++i) {
        std::cout << generator() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **选择合适的引擎**: 一般情况使用 mt19937
2. **设置种子**: 使用时间或真随机源作为种子
3. **重用引擎**: 不要频繁创建引擎对象
4. **选择合适的分布**: 根据需求选择分布类型
5. **C++11+**: 优先使用 `<random>`

---

## 参考资源

- [Boost.Random 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_random.html)
- [随机数生成器对比](https://www.boost.org/doc/libs/1_90_0/doc/html/boost_random/reference.html)

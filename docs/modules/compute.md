# Boost.Compute - GPU 计算库

## 概述

Boost.Compute 是基于 OpenCL 的 C++ GPU 计算库，提供 STL 风格的 API 用于 GPU 编程。

**类型**: 仅头文件库

**依赖**: OpenCL (需要安装 OpenCL SDK 和驱动)

---

## 快速开始

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/algorithm/transform.hpp>
#include <boost/compute/container/vector.hpp>
#include <iostream>
#include <vector>

namespace compute = boost::compute;

int main() {
    // 获取默认 GPU 设备
    compute::device gpu = compute::system::default_device();
    compute::context context(gpu);
    compute::command_queue queue(context, gpu);

    std::cout << "使用设备: " << gpu.name() << std::endl;

    // 主机数据
    std::vector<float> host_vector = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f};

    // 设备数据
    compute::vector<float> device_vector(host_vector.size(), context);

    // 复制到 GPU
    compute::copy(host_vector.begin(), host_vector.end(),
                 device_vector.begin(), queue);

    // GPU 上执行转换（每个元素乘以 2）
    compute::transform(device_vector.begin(), device_vector.end(),
                      device_vector.begin(),
                      compute::multiplies<float>(),
                      queue);

    // 复制回主机
    compute::copy(device_vector.begin(), device_vector.end(),
                 host_vector.begin(), queue);

    // 输出结果
    std::cout << "结果: ";
    for (float x : host_vector) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lOpenCL -o example
```

---

## 设备查询

```cpp
#include <boost/compute/core.hpp>
#include <iostream>

namespace compute = boost::compute;

int main() {
    // 获取所有平台
    std::vector<compute::platform> platforms = compute::system::platforms();

    std::cout << "找到 " << platforms.size() << " 个平台\n\n";

    for (size_t i = 0; i < platforms.size(); ++i) {
        compute::platform platform = platforms[i];

        std::cout << "平台 " << i << ":\n";
        std::cout << "  名称: " << platform.name() << std::endl;
        std::cout << "  供应商: " << platform.vendor() << std::endl;
        std::cout << "  版本: " << platform.version() << std::endl;

        // 获取该平台的所有设备
        std::vector<compute::device> devices = platform.devices();

        std::cout << "  设备数量: " << devices.size() << "\n\n";

        for (size_t j = 0; j < devices.size(); ++j) {
            compute::device device = devices[j];

            std::cout << "  设备 " << j << ":\n";
            std::cout << "    名称: " << device.name() << std::endl;
            std::cout << "    类型: ";

            if (device.type() & compute::device::gpu) {
                std::cout << "GPU";
            } else if (device.type() & compute::device::cpu) {
                std::cout << "CPU";
            } else {
                std::cout << "其他";
            }
            std::cout << std::endl;

            std::cout << "    计算单元: " << device.compute_units() << std::endl;
            std::cout << "    全局内存: "
                     << device.global_memory_size() / (1024 * 1024) << " MB"
                     << std::endl;
            std::cout << "    本地内存: "
                     << device.local_memory_size() / 1024 << " KB"
                     << std::endl;
            std::cout << "    最大工作组: "
                     << device.max_work_group_size() << std::endl;
            std::cout << std::endl;
        }
    }

    return 0;
}
```

---

## 向量运算

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/algorithm/transform.hpp>
#include <boost/compute/container/vector.hpp>
#include <boost/compute/functional.hpp>
#include <iostream>
#include <vector>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    // 创建设备向量
    std::vector<float> a_host = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f};
    std::vector<float> b_host = {5.0f, 4.0f, 3.0f, 2.0f, 1.0f};

    compute::vector<float> a(a_host.begin(), a_host.end(), queue);
    compute::vector<float> b(b_host.begin(), b_host.end(), queue);
    compute::vector<float> c(a.size(), ctx);

    // 向量加法: c = a + b
    compute::transform(a.begin(), a.end(), b.begin(), c.begin(),
                      compute::plus<float>(), queue);

    std::vector<float> c_host(c.size());
    compute::copy(c.begin(), c.end(), c_host.begin(), queue);

    std::cout << "a + b = ";
    for (float x : c_host) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // 向量乘法: c = a * b
    compute::transform(a.begin(), a.end(), b.begin(), c.begin(),
                      compute::multiplies<float>(), queue);

    compute::copy(c.begin(), c.end(), c_host.begin(), queue);

    std::cout << "a * b = ";
    for (float x : c_host) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 排序算法

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/algorithm/sort.hpp>
#include <boost/compute/container/vector.hpp>
#include <iostream>
#include <vector>
#include <random>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    // 生成随机数据
    std::vector<int> host_data(1000000);
    std::mt19937 gen;
    std::uniform_int_distribution<int> dist(0, 1000000);

    for (auto& x : host_data) {
        x = dist(gen);
    }

    std::cout << "排序 " << host_data.size() << " 个整数..." << std::endl;

    // 复制到 GPU
    compute::vector<int> device_data(host_data.begin(), host_data.end(), queue);

    // GPU 排序
    auto start = std::chrono::high_resolution_clock::now();
    compute::sort(device_data.begin(), device_data.end(), queue);
    queue.finish();
    auto end = std::chrono::high_resolution_clock::now();

    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << "GPU 排序耗时: " << duration.count() << " ms" << std::endl;

    // 复制回主机
    compute::copy(device_data.begin(), device_data.end(),
                 host_data.begin(), queue);

    // 验证排序结果
    bool sorted = std::is_sorted(host_data.begin(), host_data.end());
    std::cout << "排序正确: " << (sorted ? "是" : "否") << std::endl;

    return 0;
}
```

---

## 归约操作

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/algorithm/accumulate.hpp>
#include <boost/compute/algorithm/reduce.hpp>
#include <boost/compute/container/vector.hpp>
#include <iostream>
#include <vector>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    // 创建数据
    std::vector<float> host_data(1000000, 1.0f);
    compute::vector<float> device_data(host_data.begin(), host_data.end(), queue);

    // 求和
    float sum = compute::accumulate(device_data.begin(), device_data.end(),
                                   0.0f, queue);
    std::cout << "Sum: " << sum << std::endl;

    // 求最大值
    float max_val = compute::reduce(device_data.begin(), device_data.end(),
                                   compute::max<float>(), queue);
    std::cout << "Max: " << max_val << std::endl;

    // 求最小值
    float min_val = compute::reduce(device_data.begin(), device_data.end(),
                                   compute::min<float>(), queue);
    std::cout << "Min: " << min_val << std::endl;

    return 0;
}
```

---

## 自定义 OpenCL 内核

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/container/vector.hpp>
#include <iostream>
#include <vector>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    // 定义 OpenCL 内核
    const char source[] = BOOST_COMPUTE_STRINGIZE_SOURCE(
        __kernel void square(__global float* input,
                           __global float* output)
        {
            const uint i = get_global_id(0);
            output[i] = input[i] * input[i];
        }
    );

    // 编译程序
    compute::program program =
        compute::program::create_with_source(source, ctx);

    try {
        program.build();
    } catch (const compute::opencl_error& e) {
        std::cerr << "编译错误:\n"
                  << program.build_log() << std::endl;
        return 1;
    }

    // 创建内核
    compute::kernel square_kernel(program, "square");

    // 准备数据
    std::vector<float> input_host = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f};
    compute::vector<float> input(input_host.begin(), input_host.end(), queue);
    compute::vector<float> output(input.size(), ctx);

    // 设置内核参数
    square_kernel.set_arg(0, input);
    square_kernel.set_arg(1, output);

    // 执行内核
    queue.enqueue_1d_range_kernel(square_kernel, 0, input.size(), 0);

    // 读取结果
    std::vector<float> output_host(output.size());
    compute::copy(output.begin(), output.end(), output_host.begin(), queue);

    std::cout << "Results: ";
    for (float x : output_host) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## Lambda 函数

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/algorithm/transform.hpp>
#include <boost/compute/container/vector.hpp>
#include <boost/compute/lambda.hpp>
#include <iostream>
#include <vector>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    std::vector<float> host_data = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f};
    compute::vector<float> device_data(host_data.begin(), host_data.end(), queue);

    // 使用 lambda 表达式
    using compute::lambda::_1;

    // 平方
    compute::transform(device_data.begin(), device_data.end(),
                      device_data.begin(),
                      _1 * _1,
                      queue);

    compute::copy(device_data.begin(), device_data.end(),
                 host_data.begin(), queue);

    std::cout << "Squared: ";
    for (float x : host_data) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 图像处理

```cpp
#include <boost/compute/core.hpp>
#include <boost/compute/image.hpp>
#include <iostream>

namespace compute = boost::compute;

int main() {
    compute::device gpu = compute::system::default_device();
    compute::context ctx(gpu);
    compute::command_queue queue(ctx, gpu);

    // 创建图像格式
    compute::image_format format(CL_RGBA, CL_UNSIGNED_INT8);

    // 创建 2D 图像 (256x256)
    compute::image2d image(ctx, 256, 256, format);

    std::cout << "创建了 " << image.width() << "x" << image.height()
              << " 图像" << std::endl;

    // 图像处理通常需要使用 OpenCL 内核
    // 这里仅展示图像对象的创建

    return 0;
}
```

---

## 参考资源

- [Boost.Compute 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/compute/doc/html/index.html)
- [Boost.Compute GitHub](https://github.com/boostorg/compute)
- [OpenCL 官方文档](https://www.khronos.org/opencl/)

# Boost.Coroutine2 - 协程库

## 概述

Boost.Coroutine2 提供协程支持，允许函数暂停执行并稍后恢复。

**类型**: 需要编译链接的库

**链接库**: `-lboost_context` (Coroutine2 依赖 Context)

**注意**: C++20 引入了原生协程支持

---

## 快速开始

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>

typedef boost::coroutines2::coroutine<int> coro_t;

void cooperative(coro_t::push_type& yield) {
    std::cout << "协程开始" << std::endl;

    yield(1);
    std::cout << "协程恢复，第一次" << std::endl;

    yield(2);
    std::cout << "协程恢复，第二次" << std::endl;

    yield(3);
    std::cout << "协程结束" << std::endl;
}

int main() {
    coro_t::pull_type source(cooperative);

    while (source) {
        int value = source.get();
        std::cout << "主函数收到: " << value << std::endl;
        source();
    }

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_context -o example
```

---

## 生成器模式

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>

typedef boost::coroutines2::coroutine<int> coro_t;

void fibonacci(coro_t::push_type& yield) {
    int a = 0, b = 1;

    while (true) {
        yield(a);
        int next = a + b;
        a = b;
        b = next;
    }
}

int main() {
    coro_t::pull_type source(fibonacci);

    // 生成前 10 个斐波那契数
    for (int i = 0; i < 10 && source; ++i) {
        std::cout << source.get() << " ";
        source();
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 双向数据传递

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>
#include <string>

typedef boost::coroutines2::coroutine<std::string> coro_t;

void echo_server(coro_t::push_type& yield) {
    std::string message;

    while (true) {
        // 接收消息
        message = yield.get();

        if (message == "quit") {
            std::cout << "服务器退出" << std::endl;
            break;
        }

        // 回显消息
        std::cout << "服务器收到: " << message << std::endl;
        yield("Echo: " + message);
    }
}

int main() {
    coro_t::pull_type server(
        [](coro_t::push_type& yield) {
            while (yield) {
                std::string msg = yield.get();

                if (msg == "quit") {
                    break;
                }

                std::cout << "处理: " << msg << std::endl;
                yield("已处理: " + msg);
            }
        }
    );

    server("Hello");
    std::cout << "响应: " << server.get() << std::endl;
    server();

    server("World");
    std::cout << "响应: " << server.get() << std::endl;
    server();

    server("quit");

    return 0;
}
```

---

## 范围遍历

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>
#include <vector>

typedef boost::coroutines2::coroutine<int> coro_t;

void range_generator(coro_t::push_type& yield, int start, int end, int step) {
    for (int i = start; i < end; i += step) {
        yield(i);
    }
}

int main() {
    // 生成 0 到 20 的偶数
    coro_t::pull_type evens(
        [](coro_t::push_type& yield) {
            range_generator(yield, 0, 20, 2);
        }
    );

    std::cout << "偶数: ";
    for (auto value : evens) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 树遍历

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>
#include <memory>

typedef boost::coroutines2::coroutine<int> coro_t;

struct TreeNode {
    int value;
    std::shared_ptr<TreeNode> left;
    std::shared_ptr<TreeNode> right;

    TreeNode(int v) : value(v) {}
};

void inorder_traverse(coro_t::push_type& yield, std::shared_ptr<TreeNode> node) {
    if (!node) return;

    inorder_traverse(yield, node->left);
    yield(node->value);
    inorder_traverse(yield, node->right);
}

int main() {
    // 构建二叉树
    //       4
    //      / \
    //     2   6
    //    / \ / \
    //   1  3 5  7

    auto root = std::make_shared<TreeNode>(4);
    root->left = std::make_shared<TreeNode>(2);
    root->right = std::make_shared<TreeNode>(6);
    root->left->left = std::make_shared<TreeNode>(1);
    root->left->right = std::make_shared<TreeNode>(3);
    root->right->left = std::make_shared<TreeNode>(5);
    root->right->right = std::make_shared<TreeNode>(7);

    // 中序遍历
    coro_t::pull_type traversal(
        [root](coro_t::push_type& yield) {
            inorder_traverse(yield, root);
        }
    );

    std::cout << "中序遍历: ";
    for (auto value : traversal) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 异步任务模拟

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>
#include <vector>
#include <string>

typedef boost::coroutines2::coroutine<std::string> coro_t;

void async_task(coro_t::push_type& yield, const std::string& name) {
    std::cout << name << ": 开始任务" << std::endl;
    yield("步骤 1 完成");

    std::cout << name << ": 执行步骤 2" << std::endl;
    yield("步骤 2 完成");

    std::cout << name << ": 执行步骤 3" << std::endl;
    yield("步骤 3 完成");

    std::cout << name << ": 任务完成" << std::endl;
}

int main() {
    // 创建多个协程任务
    std::vector<coro_t::pull_type> tasks;

    tasks.emplace_back(
        [](coro_t::push_type& yield) {
            async_task(yield, "任务A");
        }
    );

    tasks.emplace_back(
        [](coro_t::push_type& yield) {
            async_task(yield, "任务B");
        }
    );

    // 交替执行任务（协作式多任务）
    bool all_done = false;
    while (!all_done) {
        all_done = true;

        for (auto& task : tasks) {
            if (task) {
                std::cout << "收到: " << task.get() << std::endl;
                task();
                all_done = false;
            }
        }
    }

    return 0;
}
```

---

## 数据流处理

```cpp
#include <boost/coroutine2/all.hpp>
#include <iostream>
#include <vector>

typedef boost::coroutines2::coroutine<int> coro_t;

// 数据源
void data_source(coro_t::push_type& yield) {
    for (int i = 1; i <= 10; ++i) {
        yield(i);
    }
}

// 过滤器：只保留偶数
coro_t::pull_type filter_even(coro_t::pull_type& source) {
    return coro_t::pull_type(
        [&source](coro_t::push_type& yield) {
            for (auto value : source) {
                if (value % 2 == 0) {
                    yield(value);
                }
            }
        }
    );
}

// 转换器：平方
coro_t::pull_type transform_square(coro_t::pull_type& source) {
    return coro_t::pull_type(
        [&source](coro_t::push_type& yield) {
            for (auto value : source) {
                yield(value * value);
            }
        }
    );
}

int main() {
    coro_t::pull_type source(data_source);

    // 构建处理管道
    auto filtered = filter_even(source);
    auto transformed = transform_square(filtered);

    std::cout << "结果: ";
    for (auto value : transformed) {
        std::cout << value << " ";
    }
    std::cout << std::endl;
    // 输出: 4 16 36 64 100

    return 0;
}
```

---

## 参考资源

- [Boost.Coroutine2 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/coroutine2/doc/html/index.html)
- [协程概念介绍](https://www.boost.org/doc/libs/1_90_0/libs/coroutine2/doc/html/coroutine2/intro.html)

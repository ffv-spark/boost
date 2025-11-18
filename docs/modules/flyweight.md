# Boost.Flyweight - 享元模式库

## 概述

Boost.Flyweight 实现享元设计模式，通过共享相同的对象来节省内存。

**类型**: 仅头文件库

---

## 快速开始

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    // 使用 flyweight 存储字符串
    typedef boost::flyweight<std::string> flyweight_string;

    std::vector<flyweight_string> names;

    // 添加重复的名字
    names.push_back(flyweight_string("Alice"));
    names.push_back(flyweight_string("Bob"));
    names.push_back(flyweight_string("Alice"));  // 共享第一个 Alice
    names.push_back(flyweight_string("Charlie"));
    names.push_back(flyweight_string("Bob"));    // 共享第一个 Bob

    std::cout << "存储了 " << names.size() << " 个名字\n";

    // 访问值
    for (const auto& name : names) {
        std::cout << name << " ";
    }
    std::cout << std::endl;

    // 检查是否共享
    if (&names[0].get() == &names[2].get()) {
        std::cout << "Alice 实例被共享" << std::endl;
    }

    return 0;
}
```

---

## 内存节省示例

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>
#include <vector>

int main() {
    const int count = 10000;
    std::string common_string(1000, 'x');  // 1000个字符

    // 不使用 flyweight
    {
        std::vector<std::string> vec;
        for (int i = 0; i < count; ++i) {
            vec.push_back(common_string);
        }

        size_t memory = vec.size() * sizeof(std::string) +
                       vec.size() * common_string.size();
        std::cout << "不使用 flyweight 的内存: ~"
                  << memory / 1024 << " KB" << std::endl;
    }

    // 使用 flyweight
    {
        typedef boost::flyweight<std::string> fw_string;
        std::vector<fw_string> vec;

        for (int i = 0; i < count; ++i) {
            vec.push_back(fw_string(common_string));
        }

        size_t memory = vec.size() * sizeof(fw_string) + common_string.size();
        std::cout << "使用 flyweight 的内存: ~"
                  << memory / 1024 << " KB" << std::endl;
    }

    return 0;
}
```

---

## 自定义类型

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>

struct Color {
    int r, g, b;

    Color(int r_, int g_, int b_) : r(r_), g(g_), b(b_) {}

    bool operator==(const Color& other) const {
        return r == other.r && g == other.g && b == other.b;
    }

    friend std::ostream& operator<<(std::ostream& os, const Color& c) {
        os << "RGB(" << c.r << "," << c.g << "," << c.b << ")";
        return os;
    }
};

// 为 Color 提供哈希函数
namespace std {
    template<>
    struct hash<Color> {
        size_t operator()(const Color& c) const {
            return c.r ^ (c.g << 8) ^ (c.b << 16);
        }
    };
}

int main() {
    typedef boost::flyweight<Color> flyweight_color;

    flyweight_color red(255, 0, 0);
    flyweight_color blue(0, 0, 255);
    flyweight_color red2(255, 0, 0);  // 共享

    std::cout << "红色: " << red << std::endl;
    std::cout << "蓝色: " << blue << std::endl;

    if (&red.get() == &red2.get()) {
        std::cout << "两个红色实例共享同一对象" << std::endl;
    }

    return 0;
}
```

---

## 键值分离

```cpp
#include <boost/flyweight.hpp>
#include <boost/flyweight/key_value.hpp>
#include <iostream>
#include <string>

// 文件路径和内容
struct FileContent {
    std::string path;
    std::string content;

    FileContent(const std::string& p) : path(p) {
        // 模拟从文件读取内容
        content = "Content of " + path;
    }
};

int main() {
    // 使用路径作为键，FileContent 作为值
    typedef boost::flyweight<
        boost::flyweights::key_value<std::string, FileContent>
    > flyweight_file;

    flyweight_file f1("/path/to/file.txt");
    flyweight_file f2("/path/to/file.txt");  // 共享
    flyweight_file f3("/path/to/other.txt");

    std::cout << "文件1路径: " << f1.get().path << std::endl;
    std::cout << "文件1内容: " << f1.get().content << std::endl;

    if (&f1.get() == &f2.get()) {
        std::cout << "f1 和 f2 共享同一文件内容对象" << std::endl;
    }

    return 0;
}
```

---

## 不同的工厂策略

```cpp
#include <boost/flyweight.hpp>
#include <boost/flyweight/hashed_factory.hpp>
#include <boost/flyweight/set_factory.hpp>
#include <iostream>
#include <string>

int main() {
    // 使用哈希表工厂（默认）
    typedef boost::flyweight<
        std::string,
        boost::flyweights::hashed_factory<>
    > hash_flyweight;

    hash_flyweight hfw1("Hello");
    hash_flyweight hfw2("Hello");

    // 使用 set 工厂
    typedef boost::flyweight<
        std::string,
        boost::flyweights::set_factory<>
    > set_flyweight;

    set_flyweight sfw1("World");
    set_flyweight sfw2("World");

    std::cout << "哈希工厂: " << hfw1 << std::endl;
    std::cout << "Set 工厂: " << sfw1 << std::endl;

    return 0;
}
```

---

## 持有策略

```cpp
#include <boost/flyweight.hpp>
#include <boost/flyweight/static_holder.hpp>
#include <boost/flyweight/simple_locking.hpp>
#include <iostream>
#include <string>

int main() {
    // 使用静态持有器和简单锁
    typedef boost::flyweight<
        std::string,
        boost::flyweights::static_holder,
        boost::flyweights::simple_locking
    > safe_flyweight;

    safe_flyweight fw("Thread-safe flyweight");

    std::cout << fw << std::endl;

    return 0;
}
```

---

## 标签化 flyweight

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>

struct name_tag {};
struct description_tag {};

int main() {
    // 为不同用途创建不同的 flyweight 类型
    typedef boost::flyweight<std::string, boost::flyweights::tag<name_tag>> name_fw;
    typedef boost::flyweight<std::string, boost::flyweights::tag<description_tag>> desc_fw;

    name_fw n1("Alice");
    name_fw n2("Alice");

    desc_fw d1("A developer");
    desc_fw d2("A developer");

    std::cout << "名字: " << n1 << std::endl;
    std::cout << "描述: " << d1 << std::endl;

    // n1 和 d1 即使值相同，也不共享（因为类型不同）
    return 0;
}
```

---

## 序列化支持

```cpp
#include <boost/flyweight.hpp>
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/flyweight/serialize.hpp>
#include <iostream>
#include <sstream>
#include <string>

int main() {
    typedef boost::flyweight<std::string> fw_string;

    // 创建对象
    fw_string original("Serializable flyweight");

    // 序列化
    std::stringstream ss;
    {
        boost::archive::text_oarchive oa(ss);
        oa << original;
    }

    std::cout << "序列化数据:\n" << ss.str() << std::endl;

    // 反序列化
    fw_string restored;
    {
        boost::archive::text_iarchive ia(ss);
        ia >> restored;
    }

    std::cout << "恢复的值: " << restored << std::endl;

    if (&original.get() == &restored.get()) {
        std::cout << "反序列化后共享同一对象" << std::endl;
    }

    return 0;
}
```

---

## 图形对象优化

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>
#include <vector>

// 纹理数据（通常很大）
struct Texture {
    std::string filename;
    std::vector<char> data;

    Texture(const std::string& file) : filename(file) {
        // 模拟加载纹理
        data.resize(1024 * 1024);  // 1MB
    }
};

typedef boost::flyweight<
    boost::flyweights::key_value<std::string, Texture>
> TextureFlyweight;

// 精灵对象
struct Sprite {
    TextureFlyweight texture;
    int x, y;

    Sprite(const std::string& tex_file, int x_, int y_)
        : texture(tex_file), x(x_), y(y_) {}

    void render() const {
        std::cout << "渲染精灵在 (" << x << "," << y << ") "
                  << "使用纹理: " << texture.get().filename << std::endl;
    }
};

int main() {
    std::vector<Sprite> sprites;

    // 创建多个使用相同纹理的精灵
    sprites.emplace_back("hero.png", 0, 0);
    sprites.emplace_back("hero.png", 10, 20);
    sprites.emplace_back("hero.png", 30, 40);
    sprites.emplace_back("enemy.png", 100, 50);
    sprites.emplace_back("enemy.png", 120, 60);

    std::cout << "创建了 " << sprites.size() << " 个精灵\n";
    std::cout << "但只加载了少量纹理（通过共享）\n\n";

    for (const auto& sprite : sprites) {
        sprite.render();
    }

    return 0;
}
```

---

## 统计信息

```cpp
#include <boost/flyweight.hpp>
#include <iostream>
#include <string>

int main() {
    typedef boost::flyweight<std::string> fw_string;

    fw_string s1("Hello");
    fw_string s2("World");
    fw_string s3("Hello");  // 共享 s1
    fw_string s4("Boost");
    fw_string s5("World");  // 共享 s2

    // 注意：Boost.Flyweight 没有内置的统计功能
    // 但我们可以通过检查地址来验证共享
    std::cout << "创建了 5 个 flyweight 对象\n";
    std::cout << "实际唯一值数量: 3 (Hello, World, Boost)\n";

    if (&s1.get() == &s3.get()) {
        std::cout << "s1 和 s3 共享存储" << std::endl;
    }

    if (&s2.get() == &s5.get()) {
        std::cout << "s2 和 s5 共享存储" << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.Flyweight 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/flyweight/doc/index.html)
- [享元模式介绍](https://www.boost.org/doc/libs/1_90_0/libs/flyweight/doc/tutorial/index.html)

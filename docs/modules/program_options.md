# Boost.ProgramOptions - 程序选项库

## 概述

Boost.ProgramOptions 提供强大的命令行和配置文件解析功能。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    try {
        po::options_description desc("允许的选项");
        desc.add_options()
            ("help", "显示帮助信息")
            ("version", "显示版本")
            ("input", po::value<std::string>(), "输入文件")
            ("output", po::value<std::string>(), "输出文件")
        ;

        po::variables_map vm;
        po::store(po::parse_command_line(argc, argv, desc), vm);
        po::notify(vm);

        if (vm.count("help")) {
            std::cout << desc << std::endl;
            return 0;
        }

        if (vm.count("input")) {
            std::cout << "输入文件: " << vm["input"].as<std::string>() << std::endl;
        }

    } catch (const po::error& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_program_options`

---

## 基本选项

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("help,h", "帮助")
        ("verbose,v", "详细输出")
        ("count,c", po::value<int>(), "计数")
        ("name,n", po::value<std::string>(), "名称")
    ;

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    if (vm.count("help")) {
        std::cout << desc << std::endl;
        return 0;
    }

    if (vm.count("verbose")) {
        std::cout << "详细模式启用" << std::endl;
    }

    if (vm.count("count")) {
        std::cout << "计数: " << vm["count"].as<int>() << std::endl;
    }

    if (vm.count("name")) {
        std::cout << "名称: " << vm["name"].as<std::string>() << std::endl;
    }

    return 0;
}
```

---

## 默认值

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("host", po::value<std::string>()->default_value("localhost"), "主机名")
        ("port", po::value<int>()->default_value(8080), "端口号")
        ("timeout", po::value<int>()->default_value(30), "超时（秒）")
    ;

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    std::cout << "连接配置:" << std::endl;
    std::cout << "  主机: " << vm["host"].as<std::string>() << std::endl;
    std::cout << "  端口: " << vm["port"].as<int>() << std::endl;
    std::cout << "  超时: " << vm["timeout"].as<int>() << "秒" << std::endl;

    return 0;
}
```

---

## 必需选项

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    try {
        po::options_description desc("选项");
        desc.add_options()
            ("help", "帮助")
            ("input", po::value<std::string>()->required(), "输入文件（必需）")
            ("output", po::value<std::string>()->required(), "输出文件（必需）")
        ;

        po::variables_map vm;
        po::store(po::parse_command_line(argc, argv, desc), vm);

        if (vm.count("help")) {
            std::cout << desc << std::endl;
            return 0;
        }

        po::notify(vm);  // 检查必需选项

        std::cout << "输入: " << vm["input"].as<std::string>() << std::endl;
        std::cout << "输出: " << vm["output"].as<std::string>() << std::endl;

    } catch (const po::required_option& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## 位置参数

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("input", po::value<std::string>(), "输入文件")
        ("output", po::value<std::string>(), "输出文件")
    ;

    // 定义位置参数
    po::positional_options_description p;
    p.add("input", 1);   // 第一个位置参数
    p.add("output", 1);  // 第二个位置参数

    po::variables_map vm;
    po::store(po::command_line_parser(argc, argv)
              .options(desc).positional(p).run(), vm);
    po::notify(vm);

    if (vm.count("input")) {
        std::cout << "输入: " << vm["input"].as<std::string>() << std::endl;
    }

    if (vm.count("output")) {
        std::cout << "输出: " << vm["output"].as<std::string>() << std::endl;
    }

    return 0;
}
```

---

## 配置文件

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <fstream>

namespace po = boost::program_options;

int main() {
    try {
        po::options_description desc("配置");
        desc.add_options()
            ("server.host", po::value<std::string>(), "服务器主机")
            ("server.port", po::value<int>(), "服务器端口")
            ("database.name", po::value<std::string>(), "数据库名")
            ("database.user", po::value<std::string>(), "数据库用户")
        ;

        po::variables_map vm;

        // 读取配置文件
        std::ifstream config_file("config.ini");
        if (config_file) {
            po::store(po::parse_config_file(config_file, desc), vm);
            po::notify(vm);

            std::cout << "配置:" << std::endl;
            if (vm.count("server.host")) {
                std::cout << "  服务器: " << vm["server.host"].as<std::string>()
                          << ":" << vm["server.port"].as<int>() << std::endl;
            }
            if (vm.count("database.name")) {
                std::cout << "  数据库: " << vm["database.name"].as<std::string>() << std::endl;
            }
        }

    } catch (const po::error& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

---

## 多值选项

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <vector>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("include,I", po::value<std::vector<std::string>>(), "包含路径")
    ;

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    if (vm.count("include")) {
        std::cout << "包含路径:\n";
        for (const auto& path : vm["include"].as<std::vector<std::string>>()) {
            std::cout << "  " << path << std::endl;
        }
    }

    return 0;
}

// 使用: ./program -I /usr/include -I /usr/local/include
```

---

## 选项分组

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description general("通用选项");
    general.add_options()
        ("help,h", "帮助")
        ("version,v", "版本")
    ;

    po::options_description input("输入选项");
    input.add_options()
        ("input,i", po::value<std::string>(), "输入文件")
        ("format,f", po::value<std::string>()->default_value("auto"), "输入格式")
    ;

    po::options_description output("输出选项");
    output.add_options()
        ("output,o", po::value<std::string>(), "输出文件")
        ("compress,c", "压缩输出")
    ;

    po::options_description all;
    all.add(general).add(input).add(output);

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, all), vm);
    po::notify(vm);

    if (vm.count("help")) {
        std::cout << all << std::endl;
        return 0;
    }

    return 0;
}
```

---

## 环境变量

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[], char* envp[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("home", po::value<std::string>(), "主目录")
        ("path", po::value<std::string>(), "路径")
    ;

    po::variables_map vm;

    // 从环境变量读取
    po::store(po::parse_environment(desc,
        [](const std::string& var) {
            if (var == "HOME") return "home";
            if (var == "PATH") return "path";
            return std::string();
        }), vm);

    po::notify(vm);

    if (vm.count("home")) {
        std::cout << "HOME: " << vm["home"].as<std::string>() << std::endl;
    }

    if (vm.count("path")) {
        std::cout << "PATH: " << vm["path"].as<std::string>() << std::endl;
    }

    return 0;
}
```

---

## 参考资源

- [Boost.ProgramOptions 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/program_options.html)

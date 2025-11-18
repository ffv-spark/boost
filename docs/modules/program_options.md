# Boost.Program_options - 命令行参数解析库

## 概述

Boost.Program_options 提供了强大而灵活的命令行参数和配置文件解析功能。

**类型**: 需要编译链接的库

**链接库**: `-lboost_program_options`

**主要特性**:
- 命令行参数解析
- 配置文件读取
- 自动生成帮助信息
- 类型安全
- 支持短选项和长选项
- 位置参数支持

---

## 快速开始

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    // 定义选项
    po::options_description desc("允许的选项");
    desc.add_options()
        ("help,h", "显示帮助信息")
        ("version,v", "显示版本")
        ("input,i", po::value<std::string>(), "输入文件")
        ("output,o", po::value<std::string>()->default_value("out.txt"), "输出文件")
        ("verbose", "详细输出");

    // 解析命令行
    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    // 检查选项
    if (vm.count("help")) {
        std::cout << desc << std::endl;
        return 0;
    }

    if (vm.count("verbose")) {
        std::cout << "详细模式已启用" << std::endl;
    }

    if (vm.count("input")) {
        std::cout << "输入文件: " << vm["input"].as<std::string>() << std::endl;
    }

    std::cout << "输出文件: " << vm["output"].as<std::string>() << std::endl;

    return 0;
}
```

**编译和运行**:
```bash
g++ -std=c++11 example.cpp -lboost_program_options -o myapp

./myapp --help
./myapp -i input.txt -o output.txt --verbose
./myapp --input=data.txt
```

---

## 基本用法

### 定义选项

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");

    // 1. 布尔选项（标志）
    desc.add_options()
        ("debug,d", "启用调试模式")
        ("quiet,q", "静默模式");

    // 2. 带值的选项
    desc.add_options()
        ("count,c", po::value<int>(), "计数")
        ("name,n", po::value<std::string>(), "名称")
        ("ratio,r", po::value<double>(), "比率");

    // 3. 带默认值的选项
    desc.add_options()
        ("port,p", po::value<int>()->default_value(8080), "端口号")
        ("host", po::value<std::string>()->default_value("localhost"), "主机名");

    // 4. 必需选项
    desc.add_options()
        ("config", po::value<std::string>()->required(), "配置文件（必需）");

    po::variables_map vm;
    try {
        po::store(po::parse_command_line(argc, argv, desc), vm);
        po::notify(vm);  // 会检查必需选项
    } catch (const po::error& e) {
        std::cerr << "错误: " << e.what() << std::endl;
        std::cerr << desc << std::endl;
        return 1;
    }

    // 访问选项
    if (vm.count("debug")) {
        std::cout << "调试模式" << std::endl;
    }

    if (vm.count("count")) {
        std::cout << "计数: " << vm["count"].as<int>() << std::endl;
    }

    std::cout << "端口: " << vm["port"].as<int>() << std::endl;

    return 0;
}
```

### 位置参数

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <vector>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("help", "帮助")
        ("input-file", po::value<std::vector<std::string>>(), "输入文件")
        ("output-file", po::value<std::string>(), "输出文件");

    // 定义位置参数
    po::positional_options_description pos;
    pos.add("input-file", 2);   // 前两个位置参数作为输入文件
    pos.add("output-file", 1);  // 第三个位置参数作为输出文件

    po::variables_map vm;
    po::store(po::command_line_parser(argc, argv)
                .options(desc)
                .positional(pos)
                .run(),
              vm);
    po::notify(vm);

    // 访问位置参数
    if (vm.count("input-file")) {
        std::cout << "输入文件:\n";
        for (const auto& file : vm["input-file"].as<std::vector<std::string>>()) {
            std::cout << "  - " << file << std::endl;
        }
    }

    if (vm.count("output-file")) {
        std::cout << "输出文件: " << vm["output-file"].as<std::string>() << std::endl;
    }

    return 0;
}

// 使用: ./myapp file1.txt file2.txt output.txt
```

---

## 配置文件支持

### 从配置文件读取

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <fstream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("help", "帮助")
        ("config", po::value<std::string>()->default_value("config.ini"), "配置文件");

    // 应用程序配置
    po::options_description config("配置");
    config.add_options()
        ("server.host", po::value<std::string>()->default_value("localhost"), "服务器主机")
        ("server.port", po::value<int>()->default_value(8080), "服务器端口")
        ("database.url", po::value<std::string>(), "数据库 URL")
        ("database.user", po::value<std::string>(), "数据库用户")
        ("logging.level", po::value<std::string>()->default_value("info"), "日志级别");

    po::variables_map vm;

    // 解析命令行
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    if (vm.count("help")) {
        std::cout << desc << std::endl;
        std::cout << config << std::endl;
        return 0;
    }

    // 读取配置文件
    std::string config_file = vm["config"].as<std::string>();
    std::ifstream ifs(config_file);

    if (ifs) {
        po::store(po::parse_config_file(ifs, config), vm);
        po::notify(vm);
    } else {
        std::cout << "配置文件不存在，使用默认值" << std::endl;
    }

    // 显示配置
    std::cout << "服务器配置:\n";
    std::cout << "  主机: " << vm["server.host"].as<std::string>() << std::endl;
    std::cout << "  端口: " << vm["server.port"].as<int>() << std::endl;

    if (vm.count("database.url")) {
        std::cout << "数据库配置:\n";
        std::cout << "  URL: " << vm["database.url"].as<std::string>() << std::endl;
        std::cout << "  用户: " << vm["database.user"].as<std::string>() << std::endl;
    }

    std::cout << "日志级别: " << vm["logging.level"].as<std::string>() << std::endl;

    return 0;
}
```

**配置文件示例** (`config.ini`):
```ini
# 服务器配置
server.host = 0.0.0.0
server.port = 9000

# 数据库配置
database.url = postgresql://localhost/mydb
database.user = admin

# 日志配置
logging.level = debug
```

---

## 选项分组

```cpp
#include <boost/program_options.hpp>
#include <iostream>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    // 通用选项
    po::options_description generic("通用选项");
    generic.add_options()
        ("version,v", "显示版本")
        ("help,h", "显示帮助");

    // 配置选项
    po::options_description config("配置");
    config.add_options()
        ("input,i", po::value<std::string>(), "输入文件")
        ("output,o", po::value<std::string>(), "输出文件")
        ("threads,t", po::value<int>()->default_value(4), "线程数");

    // 隐藏选项（不在帮助中显示）
    po::options_description hidden("隐藏选项");
    hidden.add_options()
        ("debug-mode", "调试模式")
        ("internal-flag", po::value<int>(), "内部标志");

    // 命令行选项 = 通用 + 配置 + 隐藏
    po::options_description cmdline_options;
    cmdline_options.add(generic).add(config).add(hidden);

    // 可见选项 = 通用 + 配置
    po::options_description visible("允许的选项");
    visible.add(generic).add(config);

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, cmdline_options), vm);
    po::notify(vm);

    if (vm.count("help")) {
        std::cout << visible << std::endl;
        return 0;
    }

    if (vm.count("version")) {
        std::cout << "版本 1.0.0" << std::endl;
        return 0;
    }

    // 处理选项...

    return 0;
}
```

---

## 高级特性

### 自定义验证器

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <regex>

namespace po = boost::program_options;

// 自定义类型：Email
struct Email {
    std::string address;
};

// 自定义验证函数
void validate(boost::any& v, const std::vector<std::string>& values, Email*, int) {
    po::validators::check_first_occurrence(v);
    const std::string& s = po::validators::get_single_string(values);

    // 验证邮箱格式
    std::regex pattern(R"(^[\w\.-]+@[\w\.-]+\.\w+$)");
    if (std::regex_match(s, pattern)) {
        v = boost::any(Email{s});
    } else {
        throw po::validation_error(po::validation_error::invalid_option_value);
    }
}

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("email", po::value<Email>(), "邮箱地址");

    po::variables_map vm;

    try {
        po::store(po::parse_command_line(argc, argv, desc), vm);
        po::notify(vm);

        if (vm.count("email")) {
            std::cout << "邮箱: " << vm["email"].as<Email>().address << std::endl;
        }
    } catch (const po::validation_error& e) {
        std::cerr << "无效的邮箱格式" << std::endl;
        return 1;
    }

    return 0;
}
```

### 多值选项

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <vector>

namespace po = boost::program_options;

int main(int argc, char* argv[]) {
    po::options_description desc("选项");
    desc.add_options()
        ("include,I", po::value<std::vector<std::string>>(), "包含路径")
        ("define,D", po::value<std::vector<std::string>>(), "定义宏")
        ("optimize,O", po::value<std::vector<int>>(), "优化级别");

    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);

    // 访问多值选项
    if (vm.count("include")) {
        std::cout << "包含路径:\n";
        for (const auto& path : vm["include"].as<std::vector<std::string>>()) {
            std::cout << "  -I" << path << std::endl;
        }
    }

    if (vm.count("define")) {
        std::cout << "宏定义:\n";
        for (const auto& macro : vm["define"].as<std::vector<std::string>>()) {
            std::cout << "  -D" << macro << std::endl;
        }
    }

    return 0;
}

// 使用: ./myapp -I/usr/include -I/usr/local/include -DDEBUG -DVERSION=2
```

---

## 实用示例

### 完整的应用程序示例

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <fstream>
#include <string>

namespace po = boost::program_options;

class Application {
public:
    bool parse_options(int argc, char* argv[]) {
        // 通用选项
        po::options_description generic("通用选项");
        generic.add_options()
            ("version,v", "显示版本信息")
            ("help,h", "显示此帮助信息")
            ("config,c", po::value<std::string>(&config_file_)->default_value("app.conf"),
             "配置文件路径");

        // 应用选项
        po::options_description app_config("应用配置");
        app_config.add_options()
            ("input,i", po::value<std::string>(&input_file_)->required(), "输入文件")
            ("output,o", po::value<std::string>(&output_file_)->default_value("output.txt"),
             "输出文件")
            ("format,f", po::value<std::string>(&format_)->default_value("text"),
             "输出格式 (text|json|xml)")
            ("threads,t", po::value<int>(&threads_)->default_value(1), "工作线程数")
            ("verbose", po::bool_switch(&verbose_), "详细输出")
            ("compression,z", po::value<int>(&compression_level_)->implicit_value(6),
             "压缩级别 (0-9)");

        po::options_description cmdline_options;
        cmdline_options.add(generic).add(app_config);

        po::options_description visible("允许的选项");
        visible.add(generic).add(app_config);

        try {
            // 解析命令行
            po::store(po::parse_command_line(argc, argv, cmdline_options), vm_);

            if (vm_.count("help")) {
                std::cout << "用法: " << argv[0] << " [选项]\n\n";
                std::cout << visible << std::endl;
                return false;
            }

            if (vm_.count("version")) {
                std::cout << "应用程序版本 1.0.0" << std::endl;
                return false;
            }

            // 读取配置文件
            std::ifstream ifs(config_file_);
            if (ifs) {
                po::store(po::parse_config_file(ifs, app_config), vm_);
            }

            po::notify(vm_);

            // 验证选项
            if (format_ != "text" && format_ != "json" && format_ != "xml") {
                throw po::error("无效的格式：" + format_);
            }

            if (threads_ < 1 || threads_ > 32) {
                throw po::error("线程数必须在 1-32 之间");
            }

        } catch (const po::error& e) {
            std::cerr << "错误: " << e.what() << "\n\n";
            std::cerr << visible << std::endl;
            return false;
        }

        return true;
    }

    void run() {
        if (verbose_) {
            std::cout << "配置:\n";
            std::cout << "  输入文件: " << input_file_ << std::endl;
            std::cout << "  输出文件: " << output_file_ << std::endl;
            std::cout << "  格式: " << format_ << std::endl;
            std::cout << "  线程数: " << threads_ << std::endl;

            if (vm_.count("compression")) {
                std::cout << "  压缩级别: " << compression_level_ << std::endl;
            }
        }

        // 执行应用逻辑...
        std::cout << "处理中..." << std::endl;
    }

private:
    po::variables_map vm_;
    std::string config_file_;
    std::string input_file_;
    std::string output_file_;
    std::string format_;
    int threads_;
    int compression_level_ = 0;
    bool verbose_ = false;
};

int main(int argc, char* argv[]) {
    Application app;

    if (app.parse_options(argc, argv)) {
        app.run();
        return 0;
    }

    return 1;
}
```

### 子命令支持

```cpp
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

namespace po = boost::program_options;

void command_add(const po::variables_map& vm) {
    std::cout << "执行 add 命令" << std::endl;
    if (vm.count("files")) {
        for (const auto& file : vm["files"].as<std::vector<std::string>>()) {
            std::cout << "  添加: " << file << std::endl;
        }
    }
}

void command_commit(const po::variables_map& vm) {
    std::cout << "执行 commit 命令" << std::endl;
    if (vm.count("message")) {
        std::cout << "  消息: " << vm["message"].as<std::string>() << std::endl;
    }
}

void command_push(const po::variables_map& vm) {
    std::cout << "执行 push 命令" << std::endl;
    if (vm.count("remote")) {
        std::cout << "  远程: " << vm["remote"].as<std::string>() << std::endl;
    }
}

int main(int argc, char* argv[]) {
    if (argc < 2) {
        std::cerr << "用法: " << argv[0] << " <command> [options]" << std::endl;
        std::cerr << "命令: add, commit, push" << std::endl;
        return 1;
    }

    std::string command = argv[1];

    // 通用选项
    po::options_description generic("通用选项");
    generic.add_options()
        ("help,h", "帮助");

    po::variables_map vm;

    if (command == "add") {
        po::options_description add_desc("add 选项");
        add_desc.add_options()
            ("files", po::value<std::vector<std::string>>(), "文件列表");

        po::positional_options_description pos;
        pos.add("files", -1);

        po::options_description all;
        all.add(generic).add(add_desc);

        po::store(po::command_line_parser(argc - 1, argv + 1)
                    .options(all).positional(pos).run(), vm);
        po::notify(vm);

        command_add(vm);

    } else if (command == "commit") {
        po::options_description commit_desc("commit 选项");
        commit_desc.add_options()
            ("message,m", po::value<std::string>()->required(), "提交消息");

        po::options_description all;
        all.add(generic).add(commit_desc);

        try {
            po::store(po::parse_command_line(argc - 1, argv + 1, all), vm);
            po::notify(vm);
            command_commit(vm);
        } catch (const po::error& e) {
            std::cerr << "错误: " << e.what() << std::endl;
            std::cerr << commit_desc << std::endl;
            return 1;
        }

    } else if (command == "push") {
        po::options_description push_desc("push 选项");
        push_desc.add_options()
            ("remote", po::value<std::string>()->default_value("origin"), "远程仓库");

        po::options_description all;
        all.add(generic).add(push_desc);

        po::store(po::parse_command_line(argc - 1, argv + 1, all), vm);
        po::notify(vm);

        command_push(vm);

    } else {
        std::cerr << "未知命令: " << command << std::endl;
        return 1;
    }

    return 0;
}

// 使用:
// ./git-clone add file1.txt file2.txt
// ./git-clone commit -m "Initial commit"
// ./git-clone push --remote=upstream
```

---

## 最佳实践

1. **总是提供帮助**: 添加 `--help` 选项
2. **使用默认值**: 为可选参数提供合理的默认值
3. **验证输入**: 在 `notify()` 后验证选项值
4. **分组选项**: 使用选项分组提高可读性
5. **配置文件**: 支持配置文件便于管理复杂配置
6. **错误处理**: 捕获并友好地显示错误信息
7. **位置参数**: 对于文件名等常用参数使用位置参数

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_program_options -o example

# CMake
find_package(Boost REQUIRED COMPONENTS program_options)
target_link_libraries(myapp Boost::program_options)
```

---

## 参考资源

- [Boost.Program_options 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/program_options.html)
- [Boost.Program_options 教程](https://www.boost.org/doc/libs/1_90_0/doc/html/program_options/tutorial.html)

# Boost.Wave - C++ 预处理器

## 概述

Boost.Wave 是一个独立的 C++ 预处理器库，可以处理 C/C++ 源代码的预处理指令。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = "#define VALUE 42\nint x = VALUE;";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "input.cpp");
    
    // 处理预处理后的输出
    std::cout << "预处理后:\\n";
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        std::cout << it->get_value();
    }
    std::cout << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_wave -lboost_thread -lboost_filesystem -lboost_system`

---

## 宏展开

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = 
        "#define ADD(a, b) ((a) + (b))\n"
        "#define MUL(a, b) ((a) * (b))\n"
        "int result = ADD(3, MUL(4, 5));";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "macro.cpp");
    
    std::cout << "展开后的代码:\\n";
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        std::cout << it->get_value();
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 条件编译

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = 
        "#define DEBUG 1\n"
        "#ifdef DEBUG\n"
        "  int debug_mode = 1;\n"
        "#else\n"
        "  int debug_mode = 0;\n"
        "#endif\n";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "conditional.cpp");
    
    std::cout << "条件编译结果:\\n";
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        std::cout << it->get_value();
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

## 包含文件

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = 
        "#include <iostream>\n"
        "int main() { return 0; }";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "include.cpp");
    
    // 添加include路径
    ctx.add_include_path("/usr/include");
    ctx.add_sysinclude_path("/usr/local/include");
    
    std::cout << "处理include后的代码（部分）:\\n";
    int count = 0;
    for (auto it = ctx.begin(); it != ctx.end() && count < 20; ++it, ++count) {
        std::cout << it->get_value();
    }
    std::cout << "..." << std::endl;
    
    return 0;
}
```

---

## 获取宏定义

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = 
        "#define PI 3.14159\n"
        "#define MAX(a, b) ((a) > (b) ? (a) : (b))\n"
        "double x = PI;";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "macros.cpp");
    
    // 预处理
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        // 处理token
    }
    
    // 获取定义的宏
    std::cout << "定义的宏:\\n";
    for (auto& macro : ctx.get_macros()) {
        std::cout << "  " << macro.first << std::endl;
    }
    
    return 0;
}
```

---

## Token 类型

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <boost/wave/token_ids.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    std::string input = "int x = 42 + 3.14;";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    context_type ctx(input.begin(), input.end(), "tokens.cpp");
    
    std::cout << "Token 分析:\\n";
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        std::cout << "  值: '" << it->get_value() << "', 类型: ";
        
        switch (it->get_id()) {
            case T_IDENTIFIER:
                std::cout << "标识符";
                break;
            case T_INTLIT:
                std::cout << "整数";
                break;
            case T_FLOATLIT:
                std::cout << "浮点数";
                break;
            case T_EQUAL:
                std::cout << "赋值";
                break;
            case T_PLUS:
                std::cout << "加号";
                break;
            default:
                std::cout << "其他";
        }
        std::cout << std::endl;
    }
    
    return 0;
}
```

---

## 错误处理

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::wave;
    
    // 包含错误的代码
    std::string input = 
        "#define X\n"
        "#if X == 1\n"  // 错误：X没有值
        "int y;\n"
        "#endif\n";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type> context_type;
    
    try {
        context_type ctx(input.begin(), input.end(), "error.cpp");
        
        for (auto it = ctx.begin(); it != ctx.end(); ++it) {
            std::cout << it->get_value();
        }
    }
    catch (const preprocess_exception& e) {
        std::cerr << "预处理错误: " << e.description() << std::endl;
        std::cerr << "文件: " << e.file_name() << std::endl;
        std::cerr << "行号: " << e.line_no() << std::endl;
    }
    
    return 0;
}
```

---

## 自定义钩子

```cpp
#include <boost/wave.hpp>
#include <boost/wave/cpplexer/cpp_lex_token.hpp>
#include <boost/wave/cpplexer/cpp_lex_iterator.hpp>
#include <iostream>
#include <string>

template <typename ContextT>
class custom_hooks : public boost::wave::context_policies::default_preprocessing_hooks {
public:
    template <typename TokenT>
    void defined_macro(ContextT const& ctx, TokenT const& name, bool is_functionlike,
                      std::vector<TokenT> const& pars,
                      boost::wave::token_sequence_type const& definition,
                      bool is_predefined) {
        std::cout << "定义宏: " << name.get_value() << std::endl;
    }
    
    template <typename TokenT>
    void expanded_macro(ContextT const& ctx, TokenT const& name) {
        std::cout << "展开宏: " << name.get_value() << std::endl;
    }
};

int main() {
    using namespace boost::wave;
    
    std::string input = 
        "#define VALUE 42\n"
        "int x = VALUE;";
    
    typedef cpplexer::lex_token<> token_type;
    typedef cpplexer::lex_iterator<token_type> lex_iterator_type;
    typedef context<std::string::iterator, lex_iterator_type,
                   iteration_context_policies::load_file_to_string,
                   custom_hooks<context<std::string::iterator, lex_iterator_type>>> context_type;
    
    context_type ctx(input.begin(), input.end(), "hooks.cpp");
    
    for (auto it = ctx.begin(); it != ctx.end(); ++it) {
        // 处理token
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Wave 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/wave/index.html)
- [预处理器教程](https://www.boost.org/doc/libs/1_90_0/libs/wave/doc/wave_driver.html)

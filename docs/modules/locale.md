# Boost.Locale - 本地化库

## 概述

Boost.Locale 提供本地化、Unicode 转换、文本处理等功能，支持多语言应用开发。

**类型**: 需要编译链接的库

**链接库**: `-lboost_locale`

---

## 快速开始

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    // 生成全局 locale
    bl::generator gen;
    std::locale::global(gen(""));

    // 使用 cout 的 locale
    std::cout.imbue(std::locale());

    // 翻译字符串（需要配置翻译文件）
    std::cout << bl::translate("Hello, World!") << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_locale -o example
```

---

## UTF-8 和编码转换

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    // UTF-8 字符串
    std::string utf8_str = "你好，世界！";

    std::cout << "UTF-8 字符串: " << utf8_str << std::endl;
    std::cout << "字节数: " << utf8_str.size() << std::endl;

    // 转换为 UTF-16
    std::u16string utf16_str = bl::conv::utf_to_utf<char16_t>(utf8_str);
    std::cout << "UTF-16 字符数: " << utf16_str.size() << std::endl;

    // 转换回 UTF-8
    std::string back_to_utf8 = bl::conv::utf_to_utf<char>(utf16_str);
    std::cout << "转换回 UTF-8: " << back_to_utf8 << std::endl;

    return 0;
}
```

---

## 文本大小写转换

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    std::string text = "Hello World";

    // 转换为大写
    std::string upper = bl::to_upper(text, loc);
    std::cout << "大写: " << upper << std::endl;

    // 转换为小写
    std::string lower = bl::to_lower(text, loc);
    std::cout << "小写: " << lower << std::endl;

    // 标题大小写（每个单词首字母大写）
    std::string title = bl::to_title(text, loc);
    std::cout << "标题: " << title << std::endl;

    // Unicode 文本
    std::string chinese = "你好世界";
    std::cout << "原文: " << chinese << std::endl;
    std::cout << "大写: " << bl::to_upper(chinese, loc) << std::endl;

    return 0;
}
```

---

## 文本规范化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    std::string text = "café";

    // NFC 规范化（标准组合形式）
    std::string nfc = bl::normalize(text, bl::norm_nfc, loc);
    std::cout << "NFC: " << nfc << " (字节数: " << nfc.size() << ")" << std::endl;

    // NFD 规范化（标准分解形式）
    std::string nfd = bl::normalize(text, bl::norm_nfd, loc);
    std::cout << "NFD: " << nfd << " (字节数: " << nfd.size() << ")" << std::endl;

    // NFKC 规范化（兼容性组合）
    std::string nfkc = bl::normalize(text, bl::norm_nfkc, loc);
    std::cout << "NFKC: " << nfkc << std::endl;

    return 0;
}
```

---

## 日期和时间格式化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <ctime>

namespace bl = boost::locale;

int main() {
    bl::generator gen;

    // 不同语言的 locale
    std::locale loc_en = gen("en_US.UTF-8");
    std::locale loc_zh = gen("zh_CN.UTF-8");
    std::locale loc_de = gen("de_DE.UTF-8");

    std::time_t now = std::time(nullptr);

    // 英文日期
    std::cout << "English: "
              << bl::as::datetime << std::use_facet<bl::date_time_facet<char>>(loc_en).format('c')
              << std::endl;

    // 使用格式化流
    std::cout.imbue(loc_en);
    std::cout << "EN: " << bl::as::datetime << now << std::endl;

    std::cout.imbue(loc_zh);
    std::cout << "ZH: " << bl::as::datetime << now << std::endl;

    std::cout.imbue(loc_de);
    std::cout << "DE: " << bl::as::datetime << now << std::endl;

    return 0;
}
```

---

## 数字格式化

```cpp
#include <boost/locale.hpp>
#include <iostream>

namespace bl = boost::locale;

int main() {
    bl::generator gen;

    std::locale loc_en = gen("en_US.UTF-8");
    std::locale loc_de = gen("de_DE.UTF-8");
    std::locale loc_zh = gen("zh_CN.UTF-8");

    double value = 12345.67;

    // 美国格式 (12,345.67)
    std::cout.imbue(loc_en);
    std::cout << "US: " << bl::as::number << value << std::endl;

    // 德国格式 (12.345,67)
    std::cout.imbue(loc_de);
    std::cout << "DE: " << bl::as::number << value << std::endl;

    // 中国格式
    std::cout.imbue(loc_zh);
    std::cout << "CN: " << bl::as::number << value << std::endl;

    // 货币格式
    std::cout.imbue(loc_en);
    std::cout << "Currency: " << bl::as::currency << value << std::endl;

    // 百分比
    std::cout << "Percent: " << bl::as::percent << 0.75 << std::endl;

    return 0;
}
```

---

## 文本排序和比较

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    std::vector<std::string> words = {
        "zebra",
        "apple",
        "café",
        "car",
        "Apple",
        "Café"
    };

    std::cout << "原始顺序:\n";
    for (const auto& w : words) {
        std::cout << "  " << w << std::endl;
    }

    // 使用 locale 排序
    std::sort(words.begin(), words.end(), bl::comparator<char>(loc));

    std::cout << "\n排序后:\n";
    for (const auto& w : words) {
        std::cout << "  " << w << std::endl;
    }

    // 不区分大小写排序
    std::sort(words.begin(), words.end(),
              bl::comparator<char>(loc, bl::collator_base::secondary));

    std::cout << "\n不区分大小写排序:\n";
    for (const auto& w : words) {
        std::cout << "  " << w << std::endl;
    }

    return 0;
}
```

---

## 边界分析（单词、句子）

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    std::string text = "Hello world! This is a test. How are you?";

    // 单词边界
    std::cout << "单词边界:\n";
    bl::boundary::ssegment_index words(
        bl::boundary::word,
        text.begin(), text.end(),
        loc
    );

    for (const auto& word : words) {
        if (word.rule() != 0) {  // 跳过空格
            std::cout << "  [" << word << "]" << std::endl;
        }
    }

    // 句子边界
    std::cout << "\n句子边界:\n";
    bl::boundary::ssegment_index sentences(
        bl::boundary::sentence,
        text.begin(), text.end(),
        loc
    );

    int i = 1;
    for (const auto& sentence : sentences) {
        std::cout << i++ << ". " << sentence << std::endl;
    }

    return 0;
}
```

---

## 字符分类

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

namespace bl = boost::locale;

int main() {
    bl::generator gen;
    std::locale loc = gen("en_US.UTF-8");

    std::string text = "Hello123 世界!";

    std::cout << "字符分析:\n";

    for (char32_t ch : bl::conv::utf_to_utf<char32_t>(text)) {
        std::cout << "字符: ";

        // 输出字符（如果可打印）
        if (bl::is_printable(ch, loc)) {
            std::string utf8_ch = bl::conv::utf_to_utf<char>(std::u32string(1, ch));
            std::cout << utf8_ch << " - ";
        } else {
            std::cout << "[不可打印] - ";
        }

        // 分类
        if (bl::is_alpha(ch, loc)) std::cout << "字母 ";
        if (bl::is_digit(ch, loc)) std::cout << "数字 ";
        if (bl::is_space(ch, loc)) std::cout << "空格 ";
        if (bl::is_punct(ch, loc)) std::cout << "标点 ";
        if (bl::is_upper(ch, loc)) std::cout << "大写 ";
        if (bl::is_lower(ch, loc)) std::cout << "小写 ";

        std::cout << std::endl;
    }

    return 0;
}
```

---

## 消息翻译

```cpp
#include <boost/locale.hpp>
#include <iostream>

namespace bl = boost::locale;

int main() {
    bl::generator gen;

    // 添加消息目录路径（需要准备 .mo 文件）
    gen.add_messages_path("./locale");
    gen.add_messages_domain("messages");

    // 设置 locale
    std::locale loc = gen("zh_CN.UTF-8");
    std::locale::global(loc);
    std::cout.imbue(loc);

    // 简单翻译
    std::cout << bl::translate("Hello") << std::endl;

    // 带上下文的翻译
    std::cout << bl::translate("menu", "File") << std::endl;

    // 复数形式
    int n = 5;
    std::cout << bl::format(bl::translate("You have {1} message", "You have {1} messages", n)) % n
              << std::endl;

    return 0;
}
```

**注意**: 翻译功能需要准备 `.po` 和 `.mo` 翻译文件。

---

## 参考资源

- [Boost.Locale 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/locale/doc/html/index.html)
- [Boost.Locale 教程](https://www.boost.org/doc/libs/1_90_0/libs/locale/doc/html/tutorials.html)

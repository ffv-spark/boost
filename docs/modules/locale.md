# Boost.Locale - 本地化库

## 概述

Boost.Locale 提供文本处理和本地化功能，支持Unicode、字符编码转换、日期/时间格式化等。

**类型**: 需要编译的库

---

## 快速开始

```cpp
#include <boost/locale.hpp>
#include <iostream>

int main() {
    using namespace boost::locale;
    
    // 生成全局locale
    generator gen;
    std::locale loc = gen("");
    std::locale::global(loc);
    std::cout.imbue(loc);
    
    // 转换大小写
    std::string text = "Hello World";
    std::cout << "原始: " << text << std::endl;
    std::cout << "大写: " << to_upper(text) << std::endl;
    std::cout << "小写: " << to_lower(text) << std::endl;
    
    return 0;
}
```

**编译**: `g++ -std=c++14 example.cpp -lboost_locale`

---

## 字符编码转换

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::locale;
    
    // UTF-8 到 UTF-16
    std::string utf8_text = "你好，世界！";
    std::wstring utf16_text = conv::utf_to_utf<wchar_t>(utf8_text);
    
    std::cout << "UTF-8 大小: " << utf8_text.size() << " 字节" << std::endl;
    std::wcout << L"UTF-16 大小: " << utf16_text.size() << L" 字符" << std::endl;
    
    // UTF-16 到 UTF-8
    std::string back_to_utf8 = conv::utf_to_utf<char>(utf16_text);
    std::cout << "转换回来: " << back_to_utf8 << std::endl;
    
    return 0;
}
```

---

## 文本规范化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale loc = gen("");
    
    std::string text = "Café";
    
    // NFD (分解)
    std::string nfd = normalize(text, norm_nfd, loc);
    std::cout << "NFD: " << nfd << " (大小: " << nfd.size() << ")" << std::endl;
    
    // NFC (组合)
    std::string nfc = normalize(text, norm_nfc, loc);
    std::cout << "NFC: " << nfc << " (大小: " << nfc.size() << ")" << std::endl;
    
    return 0;
}
```

---

## 大小写转换

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <string>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale loc = gen("en_US.UTF-8");
    
    std::string text = "the quick BROWN fox";
    
    std::cout << "原始: " << text << std::endl;
    std::cout << "大写: " << to_upper(text, loc) << std::endl;
    std::cout << "小写: " << to_lower(text, loc) << std::endl;
    std::cout << "标题: " << to_title(text, loc) << std::endl;
    
    return 0;
}
```

---

## 排序和比较

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale loc = gen("zh_CN.UTF-8");
    
    std::vector<std::string> words = {
        "张三", "李四", "王五", "赵六"
    };
    
    // 使用locale排序
    std::sort(words.begin(), words.end(), comparator<char>(loc, collator_base::primary));
    
    std::cout << "排序后:\\n";
    for (const auto& word : words) {
        std::cout << "  " << word << std::endl;
    }
    
    return 0;
}
```

---

## 数字格式化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <sstream>

int main() {
    using namespace boost::locale;
    
    generator gen;
    
    // 中文格式
    std::locale cn = gen("zh_CN.UTF-8");
    std::ostringstream cn_ss;
    cn_ss.imbue(cn);
    cn_ss << as::number << 1234567.89;
    std::cout << "中文格式: " << cn_ss.str() << std::endl;
    
    // 英文格式
    std::locale us = gen("en_US.UTF-8");
    std::ostringstream us_ss;
    us_ss.imbue(us);
    us_ss << as::number << 1234567.89;
    std::cout << "英文格式: " << us_ss.str() << std::endl;
    
    return 0;
}
```

---

## 货币格式化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <sstream>

int main() {
    using namespace boost::locale;
    
    generator gen;
    
    double amount = 1234.56;
    
    // 人民币
    std::locale cn = gen("zh_CN.UTF-8");
    std::ostringstream cn_ss;
    cn_ss.imbue(cn);
    cn_ss << as::currency << amount;
    std::cout << "人民币: " << cn_ss.str() << std::endl;
    
    // 美元
    std::locale us = gen("en_US.UTF-8");
    std::ostringstream us_ss;
    us_ss.imbue(us);
    us_ss << as::currency << amount;
    std::cout << "美元: " << us_ss.str() << std::endl;
    
    return 0;
}
```

---

## 日期时间格式化

```cpp
#include <boost/locale.hpp>
#include <iostream>
#include <ctime>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale cn = gen("zh_CN.UTF-8");
    
    std::time_t now = std::time(nullptr);
    
    std::cout.imbue(cn);
    std::cout << "短格式: " << as::date << now << std::endl;
    std::cout << "长格式: " << as::date_long << now << std::endl;
    std::cout << "完整格式: " << as::date_full << now << std::endl;
    std::cout << "时间: " << as::time << now << std::endl;
    std::cout << "日期时间: " << as::datetime << now << std::endl;
    
    return 0;
}
```

---

## 消息翻译

```cpp
#include <boost/locale.hpp>
#include <iostream>

int main() {
    using namespace boost::locale;
    
    generator gen;
    gen.add_messages_path("./locale");  // 翻译文件路径
    gen.add_messages_domain("messages");
    
    std::locale loc = gen("zh_CN.UTF-8");
    std::locale::global(loc);
    std::cout.imbue(loc);
    
    // 翻译消息
    std::cout << translate("Hello") << std::endl;
    std::cout << translate("Good morning") << std::endl;
    
    // 上下文翻译
    std::cout << translate("menu", "File") << std::endl;
    std::cout << translate("menu", "Edit") << std::endl;
    
    return 0;
}
```

---

## 复数形式

```cpp
#include <boost/locale.hpp>
#include <iostream>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale loc = gen("en_US.UTF-8");
    std::locale::global(loc);
    std::cout.imbue(loc);
    
    for (int n = 0; n <= 3; ++n) {
        std::cout << format(translate("You have {1} message", "You have {1} messages", n)) % n
                  << std::endl;
    }
    
    return 0;
}
```

---

## 边界分析

```cpp
#include <boost/locale.hpp>
#include <iostream>

int main() {
    using namespace boost::locale;
    
    generator gen;
    std::locale loc = gen("en_US.UTF-8");
    
    std::string text = "Hello world! This is a test.";
    
    // 单词边界
    boundary::ssegment_index words(boundary::word, text.begin(), text.end(), loc);
    
    std::cout << "单词:\\n";
    for (const auto& word : words) {
        if (word.rule() & boundary::word_letters) {
            std::cout << "  " << word << std::endl;
        }
    }
    
    // 句子边界
    boundary::ssegment_index sentences(boundary::sentence, text.begin(), text.end(), loc);
    
    std::cout << "\\n句子:\\n";
    for (const auto& sentence : sentences) {
        std::cout << "  " << sentence << std::endl;
    }
    
    return 0;
}
```

---

## 参考资源

- [Boost.Locale 官方文档](https://www.boost.org/doc/libs/1_90_0/libs/locale/doc/html/index.html)
- [国际化指南](https://www.boost.org/doc/libs/1_90_0/libs/locale/doc/html/tutorials.html)

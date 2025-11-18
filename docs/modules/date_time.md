# Boost.Date_Time - 日期时间库

## 概述

Boost.Date_Time 提供了全面的日期、时间和时区处理功能。

**类型**: 需要编译链接的库

**链接库**: `-lboost_date_time`

**注意**: C++11 引入了 `<chrono>`，C++20 引入了更完善的日历支持

**主要组件**:
- Gregorian Date (公历日期)
- Posix Time (POSIX 时间)
- Local Time (本地时间/时区)
- Time Duration (时间间隔)

---

## 快速开始

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>
#include <iostream>

namespace greg = boost::gregorian;
namespace pt = boost::posix_time;

int main() {
    // 日期
    greg::date today = greg::day_clock::local_day();
    std::cout << "今天: " << today << std::endl;

    // 时间
    pt::ptime now = pt::second_clock::local_time();
    std::cout << "现在: " << now << std::endl;

    // 时间间隔
    greg::date_duration week(7);
    greg::date next_week = today + week;
    std::cout << "下周: " << next_week << std::endl;

    return 0;
}
```

**编译**:
```bash
g++ -std=c++11 example.cpp -lboost_date_time -o example
```

---

## 日期处理 (Gregorian)

### 创建日期

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>

namespace greg = boost::gregorian;

int main() {
    // 1. 从年月日创建
    greg::date d1(2024, 3, 15);
    greg::date d2(2024, greg::Mar, 15);

    // 2. 从字符串创建
    greg::date d3 = greg::from_string("2024-03-15");
    greg::date d4 = greg::from_string("2024/3/15");
    greg::date d5 = greg::from_undelimited_string("20240315");

    // 3. 当前日期
    greg::date today = greg::day_clock::local_day();
    greg::date today_utc = greg::day_clock::universal_day();

    // 4. 特殊日期
    greg::date min_date = greg::date(greg::min_date_time);
    greg::date max_date = greg::date(greg::max_date_time);
    greg::date not_a_date = greg::date(greg::not_a_date_time);

    std::cout << "今天: " << today << std::endl;
    std::cout << "UTC: " << today_utc << std::endl;

    return 0;
}
```

### 日期操作

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>

namespace greg = boost::gregorian;

int main() {
    greg::date d(2024, 3, 15);

    // 1. 日期分量
    std::cout << "年: " << d.year() << std::endl;
    std::cout << "月: " << d.month() << std::endl;
    std::cout << "日: " << d.day() << std::endl;
    std::cout << "星期: " << d.day_of_week() << std::endl;  // 0=Sunday
    std::cout << "年中第几天: " << d.day_of_year() << std::endl;

    // 2. 日期算术
    greg::days one_day(1);
    greg::date tomorrow = d + one_day;
    greg::date yesterday = d - one_day;

    greg::weeks two_weeks(2);
    greg::date future = d + two_weeks;

    greg::months one_month(1);
    greg::date next_month = d + one_month;

    greg::years one_year(1);
    greg::date next_year = d + one_year;

    // 3. 日期差
    greg::date d1(2024, 1, 1);
    greg::date d2(2024, 12, 31);
    greg::date_duration diff = d2 - d1;
    std::cout << "相差天数: " << diff.days() << std::endl;

    // 4. 日期比较
    if (d1 < d2) {
        std::cout << d1 << " 早于 " << d2 << std::endl;
    }

    return 0;
}
```

### 日期范围和迭代

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>

namespace greg = boost::gregorian;

int main() {
    // 1. 日期范围
    greg::date start(2024, 3, 1);
    greg::date end(2024, 3, 31);
    greg::date_period march(start, end);

    std::cout << "三月: " << march << std::endl;
    std::cout << "天数: " << march.length().days() << std::endl;

    // 检查日期是否在范围内
    greg::date check(2024, 3, 15);
    if (march.contains(check)) {
        std::cout << check << " 在三月内" << std::endl;
    }

    // 2. 日期迭代器
    std::cout << "\n2024年3月的所有星期一:\n";
    greg::day_iterator it(start);
    while (it <= end) {
        if (it->day_of_week() == greg::Monday) {
            std::cout << *it << std::endl;
        }
        ++it;
    }

    // 3. 月迭代器
    std::cout << "\n2024年每月的第一天:\n";
    greg::month_iterator mit(greg::date(2024, 1, 1));
    for (int i = 0; i < 12; ++i) {
        std::cout << *mit << std::endl;
        ++mit;
    }

    // 4. 周迭代器
    std::cout << "\n接下来4周的星期一:\n";
    greg::week_iterator wit(start, 1);  // 每次增加1周
    for (int i = 0; i < 4; ++i) {
        std::cout << *wit << std::endl;
        ++wit;
    }

    return 0;
}
```

### 日期格式化

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>
#include <sstream>

namespace greg = boost::gregorian;

int main() {
    greg::date d(2024, 3, 15);

    // 1. 默认格式
    std::cout << "默认: " << d << std::endl;  // 2024-Mar-15

    // 2. ISO 格式
    std::cout << "ISO: " << greg::to_iso_string(d) << std::endl;  // 20240315
    std::cout << "ISO 扩展: " << greg::to_iso_extended_string(d) << std::endl;  // 2024-03-15

    // 3. SQL 格式
    std::cout << "SQL: " << greg::to_sql_string(d) << std::endl;  // 2024-03-15

    // 4. 简单格式
    std::cout << "简单: " << greg::to_simple_string(d) << std::endl;  // 2024-Mar-15

    // 5. 自定义格式（使用 facet）
    std::ostringstream oss;
    greg::date_facet* facet = new greg::date_facet("%Y年%m月%d日");
    oss.imbue(std::locale(oss.getloc(), facet));
    oss << d;
    std::cout << "自定义: " << oss.str() << std::endl;  // 2024年03月15日

    return 0;
}
```

---

## 时间处理 (Posix Time)

### 创建时间

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
#include <iostream>

namespace pt = boost::posix_time;
namespace greg = boost::gregorian;

int main() {
    // 1. 从日期和时间创建
    pt::ptime t1(greg::date(2024, 3, 15), pt::hours(10) + pt::minutes(30) + pt::seconds(45));

    // 2. 从字符串创建
    pt::ptime t2 = pt::time_from_string("2024-03-15 10:30:45");
    pt::ptime t3 = pt::from_iso_string("20240315T103045");

    // 3. 当前时间
    pt::ptime now = pt::second_clock::local_time();
    pt::ptime now_utc = pt::second_clock::universal_time();
    pt::ptime now_precise = pt::microsec_clock::local_time();

    // 4. 特殊时间
    pt::ptime epoch(greg::date(1970, 1, 1));
    pt::ptime not_a_time(pt::not_a_date_time);

    std::cout << "现在: " << now << std::endl;
    std::cout << "UTC: " << now_utc << std::endl;
    std::cout << "精确: " << now_precise << std::endl;

    return 0;
}
```

### 时间操作

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
#include <iostream>

namespace pt = boost::posix_time;
namespace greg = boost::gregorian;

int main() {
    pt::ptime t(greg::date(2024, 3, 15), pt::hours(10) + pt::minutes(30));

    // 1. 时间分量
    std::cout << "日期: " << t.date() << std::endl;
    std::cout << "时间: " << t.time_of_day() << std::endl;
    std::cout << "小时: " << t.time_of_day().hours() << std::endl;
    std::cout << "分钟: " << t.time_of_day().minutes() << std::endl;
    std::cout << "秒: " << t.time_of_day().seconds() << std::endl;

    // 2. 时间算术
    pt::time_duration one_hour = pt::hours(1);
    pt::ptime later = t + one_hour;
    pt::ptime earlier = t - one_hour;

    pt::time_duration two_and_half_hours = pt::hours(2) + pt::minutes(30);
    pt::ptime future = t + two_and_half_hours;

    // 3. 时间差
    pt::ptime t1 = pt::time_from_string("2024-03-15 10:00:00");
    pt::ptime t2 = pt::time_from_string("2024-03-15 15:30:45");
    pt::time_duration diff = t2 - t1;

    std::cout << "\n时间差:\n";
    std::cout << "总秒数: " << diff.total_seconds() << std::endl;
    std::cout << "总分钟数: " << diff.total_milliseconds() / 1000 / 60 << std::endl;
    std::cout << "格式化: " << pt::to_simple_string(diff) << std::endl;

    return 0;
}
```

### 时间Duration

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
#include <iostream>

namespace pt = boost::posix_time;

int main() {
    // 1. 创建 duration
    pt::time_duration td1 = pt::hours(2);
    pt::time_duration td2 = pt::minutes(30);
    pt::time_duration td3 = pt::seconds(45);
    pt::time_duration td4 = pt::milliseconds(500);
    pt::time_duration td5 = pt::microseconds(1000);

    // 2. 组合 duration
    pt::time_duration combined = pt::hours(1) + pt::minutes(30) + pt::seconds(45);
    std::cout << "组合: " << combined << std::endl;

    // 3. duration 算术
    pt::time_duration d1 = pt::hours(2);
    pt::time_duration d2 = pt::minutes(30);
    pt::time_duration sum = d1 + d2;
    pt::time_duration diff = d1 - d2;

    // 4. 转换
    std::cout << "总小时数: " << combined.hours() << std::endl;
    std::cout << "总秒数: " << combined.total_seconds() << std::endl;
    std::cout << "总毫秒数: " << combined.total_milliseconds() << std::endl;
    std::cout << "总微秒数: " << combined.total_microseconds() << std::endl;

    // 5. 分数小时
    pt::time_duration fractional = pt::hours(2) + pt::minutes(30);
    double hours_decimal = fractional.total_seconds() / 3600.0;
    std::cout << "小时（十进制）: " << hours_decimal << std::endl;

    return 0;
}
```

---

## 实用示例

### 计时器

```cpp
#include <boost/date_time/posix_time/posix_time.hpp>
#include <iostream>
#include <thread>
#include <chrono>

namespace pt = boost::posix_time;

class Timer {
public:
    void start() {
        start_ = pt::microsec_clock::local_time();
    }

    void stop() {
        end_ = pt::microsec_clock::local_time();
    }

    pt::time_duration elapsed() const {
        return end_ - start_;
    }

    void print_elapsed() const {
        pt::time_duration td = elapsed();
        std::cout << "耗时: "
                  << td.total_milliseconds() << " ms ("
                  << td.total_seconds() << " s)"
                  << std::endl;
    }

private:
    pt::ptime start_;
    pt::ptime end_;
};

int main() {
    Timer timer;

    timer.start();

    // 模拟一些工作
    std::this_thread::sleep_for(std::chrono::milliseconds(1500));

    timer.stop();
    timer.print_elapsed();

    return 0;
}
```

### 年龄计算

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>

namespace greg = boost::gregorian;

struct Age {
    int years;
    int months;
    int days;
};

Age calculate_age(const greg::date& birth_date) {
    greg::date today = greg::day_clock::local_day();

    Age age;
    age.years = today.year() - birth_date.year();
    age.months = today.month() - birth_date.month();
    age.days = today.day() - birth_date.day();

    if (age.days < 0) {
        age.months--;
        greg::date prev_month = today - greg::months(1);
        age.days += prev_month.end_of_month().day();
    }

    if (age.months < 0) {
        age.years--;
        age.months += 12;
    }

    return age;
}

int main() {
    greg::date birth(1990, 5, 15);
    Age age = calculate_age(birth);

    std::cout << "出生日期: " << birth << std::endl;
    std::cout << "年龄: " << age.years << " 年 "
              << age.months << " 月 "
              << age.days << " 天" << std::endl;

    return 0;
}
```

### 工作日计算

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>

namespace greg = boost::gregorian;

bool is_weekend(const greg::date& d) {
    greg::greg_weekday wd = d.day_of_week();
    return wd == greg::Saturday || wd == greg::Sunday;
}

int count_working_days(const greg::date& start, const greg::date& end) {
    int count = 0;
    greg::day_iterator it(start);

    while (it <= end) {
        if (!is_weekend(*it)) {
            count++;
        }
        ++it;
    }

    return count;
}

greg::date add_working_days(const greg::date& start, int working_days) {
    greg::date current = start;
    int added = 0;

    while (added < working_days) {
        current += greg::days(1);
        if (!is_weekend(current)) {
            added++;
        }
    }

    return current;
}

int main() {
    greg::date start(2024, 3, 1);
    greg::date end(2024, 3, 31);

    std::cout << "三月的工作日: "
              << count_working_days(start, end) << std::endl;

    greg::date today = greg::day_clock::local_day();
    greg::date deadline = add_working_days(today, 10);

    std::cout << "10个工作日后: " << deadline << std::endl;

    return 0;
}
```

### 日期范围检查

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>
#include <vector>

namespace greg = boost::gregorian;

class DateRange {
public:
    DateRange(const greg::date& start, const greg::date& end)
        : period_(start, end) {}

    bool contains(const greg::date& d) const {
        return period_.contains(d);
    }

    bool overlaps(const DateRange& other) const {
        return period_.intersects(other.period_);
    }

    DateRange intersection(const DateRange& other) const {
        greg::date_period intersect = period_.intersection(other.period_);
        return DateRange(intersect.begin(), intersect.end());
    }

    int days() const {
        return period_.length().days();
    }

    void print() const {
        std::cout << period_.begin() << " 至 " << period_.end() << std::endl;
    }

private:
    greg::date_period period_;
};

int main() {
    DateRange vacation1(greg::date(2024, 7, 1), greg::date(2024, 7, 15));
    DateRange vacation2(greg::date(2024, 7, 10), greg::date(2024, 7, 25));

    std::cout << "假期1: ";
    vacation1.print();

    std::cout << "假期2: ";
    vacation2.print();

    if (vacation1.overlaps(vacation2)) {
        std::cout << "假期重叠！" << std::endl;
        DateRange overlap = vacation1.intersection(vacation2);
        std::cout << "重叠期间: ";
        overlap.print();
    }

    return 0;
}
```

### 生成日历

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <iostream>
#include <iomanip>

namespace greg = boost::gregorian;

void print_calendar(int year, int month) {
    greg::date first_day(year, month, 1);
    greg::date last_day = first_day.end_of_month();

    std::cout << "\n    " << first_day.month().as_long_string()
              << " " << year << std::endl;
    std::cout << " 日 一 二 三 四 五 六" << std::endl;

    // 打印前导空格
    int start_dow = first_day.day_of_week();
    for (int i = 0; i < start_dow; ++i) {
        std::cout << "   ";
    }

    // 打印日期
    greg::day_iterator it(first_day);
    while (it <= last_day) {
        std::cout << std::setw(3) << it->day();

        if (it->day_of_week() == greg::Saturday) {
            std::cout << std::endl;
        }

        ++it;
    }

    std::cout << "\n" << std::endl;
}

int main() {
    print_calendar(2024, 3);
    print_calendar(2024, 4);

    return 0;
}
```

---

## 时区支持 (Local Time)

```cpp
#include <boost/date_time/local_time/local_time.hpp>
#include <iostream>

namespace lt = boost::local_time;
namespace pt = boost::posix_time;
namespace greg = boost::gregorian;

int main() {
    // 定义时区
    lt::time_zone_ptr ny_tz(
        new lt::posix_time_zone("EST-5EDT,M4.1.0,M10.5.0")
    );

    lt::time_zone_ptr tokyo_tz(
        new lt::posix_time_zone("JST+9")
    );

    // 创建本地时间
    lt::local_date_time ny_time(
        greg::date(2024, 3, 15),
        pt::hours(10),
        ny_tz,
        lt::local_date_time::NOT_DATE_TIME_ON_ERROR
    );

    std::cout << "纽约时间: " << ny_time << std::endl;

    // 转换时区
    lt::local_date_time tokyo_time = ny_time.local_time_in(tokyo_tz);
    std::cout << "东京时间: " << tokyo_time << std::endl;

    return 0;
}
```

---

## 最佳实践

1. **使用合适的精度**: 根据需求选择秒级或微秒级时钟
2. **时区处理**: 涉及时区时使用 Local Time
3. **避免隐式转换**: 明确日期和时间的创建方式
4. **异常处理**: 处理无效日期的情况
5. **性能考虑**: 频繁操作时缓存日期对象

---

## 编译选项

```bash
# 基本编译
g++ -std=c++11 example.cpp -lboost_date_time -o example

# CMake
find_package(Boost REQUIRED COMPONENTS date_time)
target_link_libraries(myapp Boost::date_time)
```

---

## 参考资源

- [Boost.Date_Time 官方文档](https://www.boost.org/doc/libs/1_90_0/doc/html/date_time.html)
- [Gregorian 日历文档](https://www.boost.org/doc/libs/1_90_0/doc/html/date_time/gregorian.html)
- [Posix Time 文档](https://www.boost.org/doc/libs/1_90_0/doc/html/date_time/posix_time.html)

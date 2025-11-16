```cpp
const auto& beijingZone = std::chrono::locate_zone("Asia/Shanghai");
const std::chrono::zoned_time now{beijingZone, std::chrono::system_clock::now()};
auto localtime = now.get_local_time();

// 输出4位数年份
std::println("{:%Y}", localtime);
// 输出2位数年份
std::println("{:%y}", localtime);
// 输出月份
std::println("{:%m}", localtime);
// 输出日期
std::println("{:%d}", localtime);
// 输出小时
std::println("{:%H}", localtime);
// 输出分钟
std::println("{:%M}", localtime);
// 输出秒（包括小数部分）
std::println("{:%S}", localtime);
// 输出秒（不包括小数部分）
std::println("{:%OS}", localtime);

// 年-月-日
std::println("{:%Y-%m-%d}", localtime);
// 时-分-秒
std::println("{:%H:%M:%OS}", localtime);

auto dp = std::chrono::floor<std::chrono::days>(localtime);  // 向下取整到"天"
auto ymd = std::chrono::year_month_day{dp};                  // 构造年月日对象
auto time = std::chrono::hh_mm_ss{localtime - dp};           // 构造时分秒对象

int year = static_cast<int>(ymd.year());              // year 类型 → int
unsigned month = static_cast<unsigned>(ymd.month());  // month 类型 → unsigned
unsigned day = static_cast<unsigned>(ymd.day());      // day 类型 → unsigned
int hour = time.hours().count();                      // hours 类型 → int
int minute = time.minutes().count();                  // minutes 类型 → int
int second = time.seconds().count();                  // seconds 类型 → int

std::println("year:{}, month:{}, days:{}, hour: {}, minute: {}, second: {}", year, month, day, hour, minute, second);
```
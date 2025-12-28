```cpp
int main() {
  std::println("|{:<10}|", "left");     // 左对齐，宽度10
  std::println("|{:>10}|", "right");    // 右对齐，宽度10
  std::println("|{:^10}|", "center");   // 居中对齐，宽度10
  std::println("|{:*^10}|", "center");  // 居中对齐，用 '*' 填充

  constexpr int num = 42;
  std::println("Decimal: {}", num);                     // 十进制
  std::println("Binary:  {:#b}", num);                  // 带前缀的二进制
  std::println("Octal:   {:#o}", num);                  // 带前缀的八进制
  std::println("Hex:     {:#x}", num);                  // 带前缀的小写十六进制
  std::println("Char:    {}", static_cast<char>(num));  // 解释为字符 '*'

  constexpr double pi = std::numbers::pi;
  constexpr double bigNum = 1.2345e30;
  std::println("Default: {}", pi);
  std::println("Fixed (2 digits): {:.2f}", pi);
  std::println("Fixed (10 digits): {:.10f}", pi);
  std::println("Scientific: {:e}", bigNum);
}
```

输出结构体：

```cpp
// 定义结构体
struct Point {
  int x;
  int y;
};

// 特化 std::formatter
template <>
struct std::formatter<Point> {
  // 解析格式字符串（这里使用默认实现）
  static constexpr auto parse(std::format_parse_context& ctx) {
    return ctx.begin();
  }

  // 格式化输出
  static auto format(const Point& p, std::format_context& ctx) {
    return std::format_to(ctx.out(), "Point({}, {})", p.x, p.y);
  }
};

int main() {
  Point p{10, 20};
  std::println("坐标: {}", p);  // 输出: 坐标: Point(10, 20)
}
```
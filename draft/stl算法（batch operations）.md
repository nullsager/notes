### std::for_each

示例 1：

```cpp
std::vector<int> v{3, -4, 2, -8, 15, 267};

auto print = [](const int& n) { std::cout << n << ' '; };

std::cout << "before:\t";
std::ranges::for_each(v, print);
std::cout << '\n';

// increment elements in-place
std::ranges::for_each(v, [](int& n) { n++; });

std::cout << "after:\t";
std::ranges::for_each(v, print);
std::cout << '\n';

struct Sum {
void operator()(int n) {
  sum += n;
}
int sum{0};
};

// invoke Sum::operator() for each element
// 写法1
Sum s = std::for_each(v.begin(), v.end(), Sum());
// 写法2
Sum s2;
//  将对象传入。因为 for_each 默认按值拷贝函数对象，
//  为了修改原始的 s_ranges，需要用 std::ref 包装一下，
//  传递其引用。
std::ranges::for_each(v, std::ref(s2));

std::cout << "sum:\t" << s.sum << '\n';
```

示例 2：

```cpp
std::vector<int> nums{3, 4, 2, 8, 15, 267};

auto print = [](const auto& n) { std::cout << ' ' << n; };

namespace ranges = std::ranges;
std::cout << "before:";
ranges::for_each(std::as_const(nums), print);
print('\n');

ranges::for_each(nums, [](int& n) { ++n; });

// calls Sum::operator() for each number
// 返回值是一个 for_each_result 对象。
// 这个对象的第一个成员 .in 将会是一个迭代器，指向 nums.end()。
// 第二个成员 .fun 将会是你最初传入的那个 Sum() 的一个副本，因此它的 sum 成员是 0。
auto [i, s] = ranges::for_each(nums.begin(), nums.end(), Sum());
assert(i == nums.end());
std::cout << s.sum << '\n';

std::cout << "after: ";
ranges::for_each(nums.cbegin(), nums.cend(), print);

std::cout << "\n"
		   "sum: "
		<< s.sum << '\n';

using Pair = std::pair<int, std::string>;
std::vector<Pair> pairs{{1, "one"}, {2, "two"}, {3, "tree"}};

std::cout << "project the pair::first: ";
ranges::for_each(pairs, print, [](const Pair& p) { return p.first; });

std::cout << "\n"
		   "project the pair::second:";
ranges::for_each(pairs, print, &Pair::second);
print('\n');
```

### std::for_each_n

第二个参数会指定要处理元素的数量。

示例 1：

```cpp
void println(auto const& v) {
  for (auto count{v.size()}; const auto& e : v) {
    std::cout << e << (--count ? ", " : "\n");
  }
}

int main() {
  std::vector<int> vi{1, 2, 3, 4, 5};
  println(vi);

  std::for_each_n(vi.begin(), 3, [](auto& n) { n *= 2; });
  println(vi);
}
```

示例 2：

```cpp
struct P {
  int first;
  char second;
  friend std::ostream& operator<<(std::ostream& os, const P& p) {
    return os << '{' << p.first << ",'" << p.second << "'}";
  }
};

auto print = [](std::string_view name, auto const& v) {
  std::cout << name << ": ";
  for (auto n = v.size(); const auto& e : v) {
    std::cout << e << (--n ? ", " : "\n");
  }
};

int main() {
  std::array a{1, 2, 3, 4, 5};
  print("a", a);
  // Negate first three numbers:
  std::ranges::for_each_n(a.begin(), 3, [](auto& n) { n *= -1; });
  print("a", a);

  std::array s{P{1, 'a'}, P{2, 'b'}, P{3, 'c'}, P{4, 'd'}};
  print("s", s);
  // Negate data members 'P::first' using projection:
  std::ranges::for_each_n(s.begin(), 2, [](auto& x) { x *= -1; }, &P::first);
  print("s", s);
  // Capitalize data members 'P::second' using projection:
  std::ranges::for_each_n(s.begin(), 3, [](auto& c) { c -= 'a' - 'A'; }, &P::second);
  print("s", s);
}
```

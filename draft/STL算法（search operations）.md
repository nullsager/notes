### all_of，any_of，none_of

```cpp
std::vector<int> v(10, 2);
// 原始的写法
// std::partial_sum(v.cbegin(), v.cend(), v.begin());
// 更现代的写法
std::inclusive_scan(v.cbegin(), v.cend(), v.begin());
std::cout << "Among the numbers: ";
// 输出vector
std::ranges::copy(v, std::ostream_iterator<int>(std::cout, " "));
std::cout << '\n';

if (std::ranges::all_of(v, [](int i) { return i % 2 == 0; })) {
  std::cout << "All numbers are even\n";
}

// && 转发引用
// 如果传入一个左值 (Lvalue)（比如一个变量 int x;），pH1 的类型会被推导为左值引用（例如 int&）。
// 如果你传入一个右值 (Rvalue)（比如一个临时值 10），pH1 的类型会被推导为右值引用（例如 int&&）。
// 任何在函数内部拥有名字的参数，其本身都是一个左值。
// 这意味着，即使 pH1 通过转发引用绑定到了一个右值，在 Lambda 的 {} 函数体内部，当你使用 pH1 这个名字时，pH1
// 本身是一个左值。 如果我们直接调用 std::modulus<>(pH1, 2)，我们传递给 std::modulus
// 的永远是一个左值，这就丢失了原始参数是“右值”这一重要信息。 对于 int
// 这样的基本类型没影响，但如果参数是一个可以被“移动”(move) 的复杂对象（如 std::string）
// 丢失右值属性会使其无法触发移动语义，从而可能导致不必要的拷贝，降低性能。

// 如果传入的是左值，decltype(pH1) 的结果是 T& (例如 int&)。
// 如果传入的是右值，decltype(pH1) 的结果是 T&& (例如 int&&)。
// std::forward<T>(arg):
// 当 T 是一个左值引用类型（int&）时，std::forward 会返回一个左值引用。
// 当 T 是一个右值引用类型（int&&）时，std::forward 会返回一个右值引用。
// std::forward<decltype(pH1)>(pH1)
// 这段代码的含义是：“请检查 pH1 最初被推导的类型，并将其转换回原始的值类别（左值或右值），然后才传递给下一个函数。
// std::modulus<>()(a, b) 的效果等同于 a % b。
if (std::ranges::none_of(v, [](auto&& pH1) { return std::modulus<>()(std::forward<decltype(pH1)>(pH1), 2); })) {
  std::cout << "None of them are odd\n";
}

struct DivisibleBy {
  int d;
  explicit DivisibleBy(int n) : d(n) {}
  bool operator()(int n) const {
    return n % d == 0;
  }
};

if (std::ranges::any_of(v, DivisibleBy(7))) {
  std::cout << "At least one number is divisible by 7\n";
}

// 另一种写法
auto divisibleBy2 = [](int d) { return [d](int m) { return m % d == 0; }; };
if (std::ranges::any_of(v, divisibleBy2(7))) {
  std::cout << "At least one number is divisible by 7\n";
}
```

### contains, contains_subrange

```cpp
int main() {
  constexpr auto haystack = std::array{3, 1, 4, 1, 5};
  constexpr auto needle = std::array{1, 4, 1};
  constexpr auto bodkin = std::array{2, 5, 2};

  // ranges::contains 检查一个范围中是否包含某个特定值。
  // ranges::contains_subrange 检查一个范围中是否包含另一个范围作为其连续的子序列。
  static_assert(ranges::contains(haystack, 4) && !ranges::contains(haystack, 6) &&
                ranges::contains_subrange(haystack, needle) && !ranges::contains_subrange(haystack, bodkin));

  constexpr std::array<std::complex<double>, 3> nums{{{1, 2}, {3, 4}, {5, 6}}};
#ifdef __cpp_lib_algorithm_default_value_type
  static_assert(ranges::contains(nums, {3, 4}));
#else
  static_assert(ranges::contains(nums, std::complex<double>{3, 4}));
#endif
}
```

### find，find_if，find_if_not

示例 1：

```cpp
bool isEven(int i) {
  return i % 2 == 0;
}

void exampleContains() {
  const auto haystack = {1, 2, 3, 4};

  for (const int needle : {3, 5}) {
    if (std::ranges::find(haystack, needle) == haystack.end()) {
      std::cout << "haystack does not contain " << needle << '\n';
    } else {
      std::cout << "haystack contains " << needle << '\n';
    }
  }
}

void examplePredicate() {
  for (const auto& haystack : {std::array{3, 1, 4}, {1, 3, 5}}) {
    const auto* const it = std::ranges::find_if(haystack, isEven);
    if (it != haystack.end()) {
      std::cout << "haystack contains an even number " << *it << '\n';
    } else {
      std::cout << "haystack does not contain even numbers\n";
    }
  }
}

void exampleListInit() {
  std::vector<std::complex<double>> haystack{{4.0, 2.0}};
#ifdef __cpp_lib_algorithm_default_value_type
  // T gets deduced making list-initialization possible
  const auto it = std::ranges::find(haystack, {4.0, 2.0});
#else
  const auto it = std::find(haystack.begin(), haystack.end(), std::complex{4.0, 2.0});
#endif
  assert(it == haystack.begin());
}

int main() {
  exampleContains();
  examplePredicate();
  exampleListInit();
}
```

示例 2：

```cpp
void projectorExample() {
  struct FolkInfo {
    unsigned uid;
    std::string name, position;
  };

  std::vector<FolkInfo> folks{{0, "Ana", "dev"}, {1, "Bob", "devops"}, {2, "Eve", "ops"}};

  const auto *const who{"Eve"};
  if (auto it = std::ranges::find(folks, who, &FolkInfo::name); it != folks.end()) {
    std::cout << std::format(
        "Profile:\n"
        "    UID: {}\n"
        "    Name: {}\n"
        "    Position: {}\n\n",
        it->uid, it->name, it->position);
  }
}

int main() {
  namespace ranges = std::ranges;

  projectorExample();

  const int n1 = 3;
  const int n2 = 5;
  const auto v = {4, 1, 3, 2};

  if (ranges::find(v, n1) != v.end()) {
    std::cout << "v contains: " << n1 << '\n';
  } else {
    std::cout << "v does not contain: " << n1 << '\n';
  }

  if (ranges::find(v.begin(), v.end(), n2) != v.end()) {
    std::cout << "v contains: " << n2 << '\n';
  } else {
    std::cout << "v does not contain: " << n2 << '\n';
  }

  auto isEven = [](int x) { return x % 2 == 0; };

  if (const auto *result = ranges::find_if(v.begin(), v.end(), isEven); result != v.end()) {
    std::cout << "First even element in v: " << *result << '\n';
  } else {
    std::cout << "No even elements in v\n";
  }

  if (const auto *result = ranges::find_if_not(v, isEven); result != v.end()) {
    std::cout << "First odd element in v: " << *result << '\n';
  } else {
    std::cout << "No odd elements in v\n";
  }

  auto divides13 = [](int x) { return x % 13 == 0; };

  if (const auto *result = ranges::find_if(v, divides13); result != v.end()) {
    std::cout << "First element divisible by 13 in v: " << *result << '\n';
  } else {
    std::cout << "No elements in v are divisible by 13\n";
  }

  if (const auto *result = ranges::find_if_not(v.begin(), v.end(), divides13); result != v.end()) {
    std::cout << "First element indivisible by 13 in v: " << *result << '\n';
  } else {
    std::cout << "All elements in v are divisible by 13\n";
  }

  std::vector<std::complex<double>> nums{{4, 2}};
#ifdef __cpp_lib_algorithm_default_value_type
  // T gets deduced in (2) making list-initialization possible
  const auto it = ranges::find(nums, {4, 2});
#else
  const auto it = ranges::find(nums, std::complex<double>{4, 2});
#endif
  assert(it == nums.begin());
}
```

### find_last,  find_last_if, find_last_if_not

```cpp
namespace ranges = std::ranges;

constexpr static auto v = {1, 2, 3, 1, 2, 3, 1, 2};

{
constexpr auto i1 = ranges::find_last(v.begin(), v.end(), 3);
constexpr auto i2 = ranges::find_last(v, 3);
static_assert(ranges::distance(v.begin(), i1.begin()) == 5);
static_assert(ranges::distance(v.begin(), i2.begin()) == 5);
}
{
constexpr auto i1 = ranges::find_last(v.begin(), v.end(), -3);
constexpr auto i2 = ranges::find_last(v, -3);
static_assert(i1.begin() == v.end());
static_assert(i2.begin() == v.end());
}

auto abs = [](int x) { return x < 0 ? -x : x; };

{
auto pred = [](int x) { return x == 3; };
// 从后往前寻找第一个绝对值为3的元素
constexpr auto i1 = ranges::find_last_if(v.begin(), v.end(), pred, abs);
constexpr auto i2 = ranges::find_last_if(v, pred, abs);
static_assert(ranges::distance(v.begin(), i1.begin()) == 5);
static_assert(ranges::distance(v.begin(), i2.begin()) == 5);
}
{
auto pred = [](int x) { return x == -3; };
constexpr auto i1 = ranges::find_last_if(v.begin(), v.end(), pred, abs);
constexpr auto i2 = ranges::find_last_if(v, pred, abs);
static_assert(i1.begin() == v.end());
static_assert(i2.begin() == v.end());
}

{
auto pred = [](int x) { return x == 1 or x == 2; };
constexpr auto i1 = ranges::find_last_if_not(v.begin(), v.end(), pred, abs);
constexpr auto i2 = ranges::find_last_if_not(v, pred, abs);
static_assert(ranges::distance(v.begin(), i1.begin()) == 5);
static_assert(ranges::distance(v.begin(), i2.begin()) == 5);
}
{
auto pred = [](int x) { return x == 1 or x == 2 or x == 3; };
constexpr auto i1 = ranges::find_last_if_not(v.begin(), v.end(), pred, abs);
constexpr auto i2 = ranges::find_last_if_not(v, pred, abs);
static_assert(i1.begin() == v.end());
static_assert(i2.begin() == v.end());
}

using P = std::pair<std::string_view, int>;
std::forward_list<P> list{
  {"one", 1}, {"two", 2}, {"three", 3}, {"one", 4}, {"two", 5}, {"three", 6},
};
auto cmpOne = [](const std::string_view& s) { return s == "one"; };

// find latest element that satisfy the comparator, and projecting pair::first
const auto subrange = ranges::find_last_if(list, cmpOne, &P::first);

std::cout << "The found element and the tail after it are:\n";
for (const P& e : subrange) {
std::cout << '{' << std::quoted(e.first) << ", " << e.second << "} ";
}
std::cout << '\n';

#if __cpp_lib_algorithm_default_value_type
const auto i3 = ranges::find_last(list, {"three", 3});  // (2) C++26
#else
const auto i3 = ranges::find_last(list, P{"three", 3});  // (2) C++23
#endif
assert(i3.begin()->first == "three" && i3.begin()->second == 3);
```

### find_end

示例 1：

```cpp
const auto printResult = [](auto result, const auto& v) {
  result == v.end() ? std::cout << "Sequence not found\n"
                    : std::cout << "Last occurrence is at: " << std::distance(v.begin(), result) << '\n';
};
const auto v = {1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4};

for (auto const& x : {std::array{1, 2, 3}, {4, 5, 6}}) {
  const auto iter = std::ranges::find_end(v, x);
  printResult(iter.begin(), v);
}

for (auto const& x : {std::array{-1, -2, -3}, {-4, -5, -6}}) {
  auto iter = std::ranges::find_end(v, x, [](int x, int y) { return std::abs(x) == std::abs(y); });
  printResult(iter.begin(), v);
}
```

示例 2：

```cpp
void print(const auto haystack, const auto needle) {
  const auto pos = std::distance(haystack.begin(), needle.begin());
  std::cout << "In \"";
  for (const auto c : haystack) {
    std::cout << c;
  }
  std::cout << "\" found \"";
  for (const auto c : needle) {
    std::cout << c;
  }
  std::cout << "\" at position [" << pos << ".." << pos + needle.size() << ")\n"
            << std::string(4 + pos, ' ') << std::string(needle.size(), '^') << '\n';
}

int main() {
  using namespace std::literals;
  constexpr auto secret{"password password word..."sv};
  constexpr auto wanted{"password"sv};

  constexpr auto found1 = std::ranges::find_end(secret.cbegin(), secret.cend(), wanted.cbegin(), wanted.cend());
  print(secret, found1);

  constexpr auto found2 = std::ranges::find_end(secret, "word"sv);
  print(secret, found2);

  const auto found3 =
      std::ranges::find_end(secret, "ORD"sv, [](const char x, const char y) {  // uses a binary predicate
        return std::tolower(x) == std::tolower(y);
      });
  print(secret, found3);

  const auto found4 = std::ranges::find_end(secret, "SWORD"sv, {}, {},
                                            [](char c) { return std::tolower(c); });  // projects the 2nd range
  print(secret, found4);

  static_assert(std::ranges::find_end(secret, "PASS"sv).empty());  // => not found
```

### find_first_of

示例 1：

```cpp
const auto printSequence = [](const auto id, const auto& seq, int pos = -1) {
  std::cout << id << "{ ";
  for (int i{}; auto const& e : seq) {
    const bool mark{i == pos};
    std::cout << (i++ ? ", " : "");
    std::cout << (mark ? "[ " : "") << e << (mark ? " ]" : "");
  }
  std::cout << " }\n";
};
const std::vector<int> v{0, 2, 3, 25, 5};
const auto t1 = {19, 10, 3, 4};
const auto t2 = {1, 6, 7, 9};

auto findAnyOf = [&printSequence](const auto& v, const auto& t) {
  const auto result = std::find_first_of(v.begin(), v.end(), t.begin(), t.end());
  if (result == v.end()) {
    std::cout << "No elements of v are equal to any element of ";
    printSequence("t = ", t);
    printSequence("v = ", v);
  } else {
    const auto pos = std::distance(v.begin(), result);
    std::cout << "Found a match (" << *result << ") at position " << pos;
    printSequence(", where t = ", t);
    printSequence("v = ", v, pos);
  }
};

findAnyOf(v, t1);
findAnyOf(v, t2);
```

示例 2：

```cpp
namespace rng = std::ranges;

constexpr static auto haystack = {1, 2, 3, 4};
constexpr static auto needles = {0, 3, 4, 3};

constexpr auto found1 = rng::find_first_of(haystack.begin(), haystack.end(), needles.begin(), needles.end());
static_assert(std::distance(haystack.begin(), found1) == 2);

constexpr auto found2 = rng::find_first_of(haystack, needles);
static_assert(std::distance(haystack.begin(), found2) == 2);

constexpr static auto negatives = {-6, -3, -4, -3};
constexpr auto notFound = rng::find_first_of(haystack, negatives);
static_assert(notFound == haystack.end());

constexpr auto found3 = rng::find_first_of(haystack, negatives, [](int x, int y) { return x == -y; });
static_assert(std::distance(haystack.begin(), found3) == 2);

struct P {
  int x, y;
};
constexpr static auto p1 = {P{1, -1}, P{2, -2}, P{3, -3}, P{4, -4}};
constexpr static auto p2 = {P{5, -5}, P{6, -3}, P{7, -5}, P{8, -3}};

// Compare only P::y data members by projecting them:
const auto *const found4 = rng::find_first_of(p1, p2, {}, &P::y, &P::y);
std::cout << "First equivalent element {" << found4->x << ", " << found4->y << "} was found at position "
          << std::distance(p1.begin(), found4) << ".\n";
```

### adjacent_find

示例 1：

```cpp
std::vector<int> v1{0, 1, 2, 3, 40, 40, 41, 41, 5};

// 查找相邻的相等元素
auto i1 = std::ranges::adjacent_find(v1);

if (i1 == v1.end()) {
  std::cout << "No matching adjacent elements\n";
} else {
  std::cout << "The first adjacent pair of equal elements is at " << std::distance(v1.begin(), i1)
            << ", *i1 = " << *i1 << '\n';
}

auto i2 = std::ranges::adjacent_find(v1, std::greater<>());
if (i2 == v1.end()) {
  std::cout << "The entire vector is sorted in ascending order\n";
} else {
  std::cout << "The last element in the non-decreasing subsequence is at " << std::distance(v1.begin(), i2)
            << ", *i2 = " << *i2 << '\n';
}
```


### 递归 lambda

```cpp
int n = 10;
auto dfs = [&](auto& self, int i) -> void {  // 1. 定义泛型lambda，接受自身引用和int
    if (i == n) return;
    else self(self, i + 1);                  // 3. 递归时传递self（即dfs自身）
};
dfs(dfs, 1);                                 // 2. 首次调用，传入dfs自身和初始值i=1
```

C++23 写法：

```cpp
int n = 10;
// 返回类型必须显式写明
auto dfs = [&](this auto&& self, int n) -> void {
  if (n > 0) {
    std::println("{}", n);
    n--;
    self(n);
  } else {
    return;
  }
};
dfs(n);
```

### Lambda 表达式的基本语法

一个完整的 Lambda 表达式的通用语法如下：

```cpp
[capture-list] (parameter-list) mutable exception-specification -> return-type {
    // 函数体 (function body)
}
```

我们来逐个分解这个复杂的结构：

1.  **`[capture-list]` (捕获列表)**：这是 Lambda 表达式的灵魂所在，它定义了 Lambda 函数体内部如何访问外部作用域（enclosing scope）的变量。这是普通函数不具备的能力。
2.  **`(parameter-list)` (参数列表)**：和普通函数的参数列表一样。可以省略括号，如果你的 Lambda 不需要参数。
3.  **`mutable` (可选)**：默认情况下，通过值捕获的变量在 Lambda 内部是 `const` 的。使用 `mutable` 关键字可以让你在 Lambda 函数体内修改这些按值捕获的变量。
4.  **`exception-specification` (可选, C++17 后不推荐)**：类似于 `noexcept`，用来指明该 Lambda 是否会抛出异常。
5.  **`-> return-type` (可选, 尾随返回类型)**：用于显式指定 Lambda 的返回类型。在大多数情况下，编译器可以根据函数体中的 `return` 语句自动推断出返回类型，所以这个部分通常可以省略。
6.  **`{ function body }` (函数体)**：和普通函数的函数体一样，包含了具体的执行代码。

在最简单的情况下，一个 Lambda 可以是这样的：

```cpp
[]() { std::cout << "Hello, Lambda!" << std::endl; }
```

下面我们来深入探讨每个部分。

-----

### 1\. `[capture-list]` 捕获列表详解

捕获列表是 Lambda 与普通函数最大的区别。它决定了 Lambda 如何“捕获”或“记住”其创建时所在作用域的变量。

| 捕获方式 | 语法 | 描述 |
| :--- | :--- | :--- |
| **不捕获任何变量** | `[]` | Lambda 内部不能访问外部作用域的任何变量。 |
| **按值捕获** | `[=]` | 以**传值**的方式捕获外部作用域的所有自动变量（局部变量）。在 Lambda 内部，这些变量是只读的（`const`），除非你使用了 `mutable` 关键字。 |
| **按引用捕获** | `[&]` | 以**传引用**的方式捕获外部作用域的所有自动变量。在 Lambda 内部可以修改这些变量，并且会影响到外部的原始变量。 |
| **指定变量按值捕获**| `[var]` | 只按值捕获名为 `var` 的变量。 |
| **指定变量按引用捕获**| `[&var]` | 只按引用捕获名为 `var` 的变量。 |
| **混合捕获** | `[=, &var]` | 默认按值捕获所有变量，但 `var` 变量按引用捕获。 |
| **混合捕获** | `[&, var]` | 默认按引用捕获所有变量，但 `var` 变量按值捕获。 |
| **捕获 `this` 指针** | `[this]` | (仅在类成员函数中) 捕获当前对象的 `this` 指针，从而可以在 Lambda 内部访问类的成员变量和成员函数。 |

#### C++14：广义捕获 (Generalized Capture) / 初始化捕获 (Init Capture)

C++14 引入了一个更强大的捕获方式，允许你在捕获列表中创建新的变量，这个新变量只在 Lambda 内部可见。这对于移动（move）一个只能移动的对象（如 `std::unique_ptr`）或者在捕获时进行计算非常有用。

语法：`[identifier = expression]` 或 `[&identifier = expression]`

**示例 1：移动所有权**

```cpp
auto ptr = std::make_unique<int>(10);
// 错误：unique_ptr 不能被拷贝
// auto myLambda = [ptr]() { /* ... */ };

// 正确：使用广义捕获来移动 ptr 的所有权
auto myLambda = [p = std::move(ptr)]() {
    std::cout << "Value from moved pointer: " << *p << std::endl;
};
myLambda();
// 此时 ptr 已经是 nullptr 了
```

**示例 2：捕获时计算**

```cpp
int x = 5, y = 10;
auto myLambda = [z = x + y]() {
    std::cout << "z = " << z << std::endl; // z 是一个值为 15 的新变量
};
myLambda();
```

-----

### 2\. `(parameter-list)` 参数列表详解

这部分和普通函数的参数列表行为一致。

```cpp
auto add = [](int a, int b) {
    return a + b;
};
int sum = add(3, 4); // sum is 7
```

#### C++14：泛型 Lambda (Generic Lambda)

C++14 允许在参数列表中使用 `auto` 关键字，这使得 Lambda 变成了一个模板。编译器会根据调用时传入的参数类型自动推导。

```cpp
auto generic_add = [](auto a, auto b) {
    return a + b;
};

int i_sum = generic_add(3, 4);           // a, b 推导为 int
double d_sum = generic_add(3.5, 4.5);   // a, b 推导为 double
std::string s_cat = generic_add(std::string("hello"), std::string(" world")); // a, b 推导为 std::string
```

#### C++20：带模板参数的泛型 Lambda

C++20 进一步增强了泛型 Lambda，允许像函数模板一样显式声明模板参数。

```cpp
auto perfect_forward = []<typename T>(T&& arg) {
    // 使用 std::forward 完美转发 arg
    return some_function(std::forward<T>(arg));
};
```

-----

### 3\. `mutable` 关键字

默认情况下，通过 `[=]` 或 `[var]` 按值捕获的变量在 Lambda 内部是 `const` 的。如果你想修改这个捕获的变量的**副本**（注意，不会影响外部原始变量），你需要使用 `mutable`。

```cpp
int counter = 0;
auto incrementer = [counter]() mutable {
    counter++; // 如果没有 mutable, 这里会编译错误
    std::cout << "Inside lambda: " << counter << std::endl;
};

incrementer(); // 输出: Inside lambda: 1
incrementer(); // 输出: Inside lambda: 2
std::cout << "Outside lambda: " << counter << std::endl; // 输出: Outside lambda: 0 (外部变量不受影响)
```

-----

### 4\. `-> return-type` 尾随返回类型

大多数情况下，编译器可以自动推断 Lambda 的返回类型。但以下情况需要显式指定：

1.  Lambda 函数体中有多个 `return` 语句，并且它们返回的类型不同。
2.  你想返回一个引用。
3.  函数体为空，或者无法从 `return` 语句推断出类型。

<!-- end list -->

```cpp
// 自动推断返回类型为 int
auto f1 = [](int x) { return x * 2; };

// 显式指定返回类型为 double
auto f2 = [](int x) -> double {
    if (x > 0) {
        return static_cast<double>(x);
    }
    return 0.0; // 如果没有显式指定，编译器可能在推断时遇到困难
};
```

-----

### 5\. C++20 及之后的新特性

#### `consteval` 和 `constexpr`

你可以在 Lambda 表达式上使用 `consteval` (C++20) 和 `constexpr` (C++17) 来确保其可以在编译期求值。

```cpp
// C++17
auto squared_constexpr = [](int n) constexpr {
    return n * n;
};
static_assert(squared_constexpr(4) == 16);

// C++20
consteval auto make_greeter(const char* name) {
    return [name]() {
        return std::string("Hello, ") + name;
    };
}
constexpr auto greeter = make_greeter("World");
static_assert(greeter() == "Hello, World");
```

#### 无状态 Lambda (Stateless Lambda)

不捕获任何变量的 Lambda (`[]`) 被称为无状态 Lambda。从 C++11 开始，它们可以被隐式转换成一个函数指针。

```cpp
void (*ptr_func)(int) = [](int x) {
    std::cout << x << std::endl;
};
ptr_func(100);
```

#### `static` 关键字 (C++23)

C++23 允许在 Lambda 内部声明 `static` 变量，其行为和在普通函数中一样。更重要的是，你可以在 Lambda 的捕获列表前使用 `static`，这使得整个 Lambda 对象都具有静态存储期，从而避免了在循环等场景下重复构造 Lambda 对象的开销。

```cpp
// C++23
void foo() {
    for (int i = 0; i < 10; ++i) {
        // 'static' 关键字确保 lambda 对象只被创建一次
        auto myLambda = static [] {
            std::cout << "Lambda created!" << std::endl;
        };
        myLambda();
    }
}
```

### 总结与实践建议

  * **优先使用 `[=]` 或 `[&]`**：明确你的意图。如果你只需要读取外部变量，使用 `[=]` 更安全。如果你需要修改外部变量，使用 `[&]`。
  * **警惕悬空引用**：当使用 `[&]` 捕获时，要确保 Lambda 的生命周期不会超过被捕获变量的生命周期。否则，Lambda 会持有一个无效的引用，导致未定义行为。
  * **善用泛型 Lambda**：`auto` 参数让 Lambda 变得非常灵活，可以大大减少模板代码的编写。
  * **利用广义捕获**：当需要移动对象（如 `unique_ptr`）或进行预计算时，广义捕获是你的最佳选择。
  * **保持 Lambda 简短**：Lambda 的设计初衷是为了编写简短、局部的函数。如果你的 Lambda 函数体变得很长很复杂，最好还是将它重构成一个独立的命名函数或函数对象。
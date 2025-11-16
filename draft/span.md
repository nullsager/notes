#cpp

![](Pasted%20image%2020250912221223.png)
## `std::span`：现代 C++中的安全、高效视图

`std::span` 是 C++20 标准库中引入的一个重要新特性，它提供了一种**轻量级、非拥有**（non-owning）的方式来引用一个连续的内存块。你可以把它想象成一个“视图”或者“窗口”，通过它能看到并操作一片数据，但这片数据本身的所有权归属于其他对象（例如 `std::vector`、`std::array` 或者一个 C 风格数组）。

`std::span` 的核心思想是**将数据的所有权和数据的访问分离开来**。它的内部实现通常只包含一个指向数据开头的指针和一个表示元素数量的大小，因此创建和传递 `std::span` 的成本极低，与传递一个指针和一个整数相当。

### 为什么需要 `std::span`？

在 `std::span` 出现之前，向函数传递一个连续序列通常有两种方式：

1.  **传递指针和大小**：这是 C 语言的传统做法。

    ```cpp
    void process_data(int* data, size_t size);
    ```

    这种方式的主要问题是：

      * **不安全**：很容易在传递指针和大小时出错，比如大小计算错误，导致缓冲区溢出。
      * **不便**：调用者需要手动管理指针和大小两个变量。
      * **表达力弱**：无法直接利用 C++的迭代器等特性。

2.  **通过引用传递容器**：

    ```cpp
    void process_data(const std::vector<int>& data);
    ```

    这种方式更安全，但不够灵活。如果你的数据存储在 C 风格数组 `int arr[]` 或者 `std::array` 中，你就无法直接将它传递给这个函数，必须先将其拷贝到一个 `std::vector` 中，造成不必要的性能开销。

`std::span` 完美地解决了以上问题。它统一了对不同来源的连续内存的访问方式，同时保证了安全性和高效性。

### `std::span` 的核心优势

1.  **灵活性和统一性**：函数参数使用 `std::span`，使得该函数可以接受来自 `std::vector`、`std::array`、C 风格数组，甚至是栈上分配的数组的数据，而无需进行任何数据拷贝。

2.  **安全性**：`std::span` 内部同时包含了指针和大小信息。这使得基于范围的 for 循环和标准库算法可以安全地在其上操作，避免了因忘记传递大小或大小错误而导致的越界访问。

3.  **高效性**：作为一个视图，`std::span` 本身不拥有数据，不参与内存分配和释放。它的创建、拷贝和传递都非常廉价，开销极小。

4.  **表现力强**：`std::span` 提供了类似标准库容器的接口，如 `size()`、`front()`、`back()`、`empty()` 以及迭代器支持（`begin()`, `end()`），使得代码更具表现力且易于理解。

### 如何使用 `std::span`

要使用 `std::span`，需要包含头文件 `<span>`。

#### 创建 `std::span`

你可以从各种连续内存容器中创建 `std::span`：

```cpp
#include <span>
#include <iostream>
#include <vector>
#include <array>

// 一个接受 span 的通用函数
void print_span(std::span<const int> s) {
    std::cout << "Span size: " << s.size() << " -> [ ";
    for (int val : s) {
        std::cout << val << " ";
    }
    std::cout << "]" << std::endl;
}

int main() {
    // 1. 从 C 风格数组创建
    int c_array[] = {1, 2, 3, 4, 5};
    std::span<int> span_from_c_array(c_array);
    print_span(span_from_c_array);

    // 2. 从 std::vector 创建
    std::vector<int> vec = {6, 7, 8, 9};
    std::span<int> span_from_vector(vec);
    print_span(span_from_vector);

    // 3. 从 std::array 创建
    std::array<int, 3> std_array = {10, 11, 12};
    std::span<int> span_from_std_array(std_array);
    print_span(span_from_std_array);

    // 使用类模板参数推导 (CTAD)，可以省略模板参数
    std::span span_auto(c_array); // 自动推导为 std::span<int, 5>
    print_span(span_auto);
}
```

**输出：**

```
Span size: 5 -> [ 1 2 3 4 5 ]
Span size: 4 -> [ 6 7 8 9 ]
Span size: 3 -> [ 10 11 12 ]
Span size: 5 -> [ 1 2 3 4 5 ]
```

#### 修改通过 `std::span` 引用的数据

`std::span` 是一个视图，通过它对元素的修改会直接反映在底层的数据容器上。如果希望 `span` 是只读的，应该使用 `std::span<const T>`。

```cpp
void double_elements(std::span<int> s) {
    for (int& val : s) {
        val *= 2;
    }
}

int main() {
    int data[] = {1, 2, 3};
    std::span<int> s(data);
    double_elements(s);

    // 底层数组的值已经被修改
    for (int val : data) {
        std::cout << val << " "; // 输出: 2 4 6
    }
    std::cout << std::endl;
}
```

#### 创建子视图 (Subspan)

`std::span` 提供了创建子视图的方法，这在处理数据流或进行算法操作时非常有用，且同样是零开销的。

  * `first(count)`：获取前 `count` 个元素的子视图。
  * `last(count)`：获取后 `count` 个元素的子视图。
  * `subspan(offset, count)`：从 `offset` 位置开始，获取 `count` 个元素的子视图。

<!-- end list -->

```cpp
#include <span>
#include <iostream>

void print_span(std::span<const int> s); // 复用上面的打印函数

int main() {
    int data[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    std::span<const int> s(data);

    // 获取前4个元素
    auto first_four = s.first(4);
    print_span(first_four); // 输出: Span size: 4 -> [ 0 1 2 3 ]

    // 获取后3个元素
    auto last_three = s.last(3);
    print_span(last_three); // 输出: Span size: 3 -> [ 7 8 9 ]

    // 从索引2开始，获取5个元素
    auto middle_part = s.subspan(2, 5);
    print_span(middle_part); // 输出: Span size: 5 -> [ 2 3 4 5 6 ]
}
```

### 注意事项和最佳实践

1.  **生命周期管理**：`std::span` 不拥有它所引用的数据。你必须确保 `std::span` 的生命周期不会超过其引用的底层数据容器的生命周期。如果底层容器被销毁，`span` 将会变成一个**悬垂引用 (dangling reference)**，使用它会导致未定义行为。

    ```cpp
    std::span<int> dangling_span;
    {
        std::vector<int> vec = {1, 2, 3};
        dangling_span = vec;
    } // vec 在这里被销毁
    // 此后使用 dangling_span 是危险的！
    // std::cout << dangling_span.front(); // 未定义行为
    ```

2.  **函数参数传递**：将 `std::span` 作为函数参数时，应该**按值传递**。因为它本身很小（通常只有两个指针大小），按值传递比按引用传递更高效。

3.  **常量正确性**：如果函数不应该修改数据，请使用 `std::span<const T>` 来接收参数，这是一种良好的编程实践。

### 总结

`std::span` 是现代 C++中一个强大且实用的工具，它极大地改善了处理连续内存数据的方式。通过提供一个统一、安全且高效的视图，它简化了 API 设计，减少了代码中的常见错误，并提高了代码的灵活性和可读性。在任何需要处理（但不拥有）连续数据序列的场景下，都应该优先考虑使用 `std::span`。
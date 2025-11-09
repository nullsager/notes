main 函数不一定时第一个执行的函数，全局变量在执行 main 之前被初始化。如果该变量的初始化器调用了一个函数，那么该函数将在 main 之前执行。

考虑如下代码：

```cpp
int f() {
  std::cout << "首个被执行\n";
  return 10;
}
int x = f();
int main() {
  std::cout << "第二个被执行\n";
}
```
---

一旦预处理器完成，来自该文件的所有定义标识符将被丢弃。这意味着指令仅在定义点到定义它们的文件末尾之间有效。在一个文件中定义的指令对其他文件没有任何影响（除非它们被#included 到另一个文件中）。例如：

function.cpp:

```cpp
#include <iostream>

void doSomething()
{
#ifdef PRINT
    std::cout << "Printing!\n";
#endif
#ifndef PRINT
    std::cout << "Not printing!\n";
#endif
}
```

main.cpp:

```cpp
void doSomething();
#define PRINT
int main() {
    doSomething();
}
```

上述程序将打印：

Not printing!

尽管 PRINT 在 main.cpp 中被定义，但这对 function.cpp 中的任何代码没有影响（PRINT 仅在定义点到 main.cpp 的末尾之间被#定义）。当我们在未来的课程中讨论头文件保护时，这将是一个重要的内容。

---


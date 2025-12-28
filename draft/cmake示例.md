指定编译器：`cmake -Bbuild -DCMAKE_CXX_COMPILER=clang++`

在运行 cmake 配置命令时，添加 `-D CMAKE_BUILD_TYPE=<Type>` 选项。

常见的构建类型：
+ Debug: 包含调试信息 (-g)，不做优化 (-O0)。最适合调试
+ Release: 开启高级别优化（如 -O3），不包含调试信息。用于最终发布的产品
+ RelWithDebInfo: （推荐用于开发） 开启优化（如 -O2），并且包含调试信息 (-g)。既有良好性能，又可以调试
+ MinSizeRel: 开启优化以尽可能减小可执行文件的大小（如 -Os）。

样例：

```cmake
cmake_minimum_required(VERSION 3.25)
project(demo VERSION 1.0)

# Set C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR})

# Add executable
add_executable(a.out
    main.cpp
)
```

---

使用`#include <>`和`#include ""`的区别是前者会自动在`/usr/include`目录下查找


`target_compile_options (a.out PRIVATE -Wall -Wextra -Wpedantic)`

开启警告提示

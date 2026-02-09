#cpp 

cmake 示例：

```cmake
cmake_minimum_required(VERSION 3.19..4.2.3)
project(app LANGUAGES CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 17)

set(CMAKE_AUTOMOC ON) # 自动处理 Q_OBJECT 宏
set(CMAKE_AUTOUIC ON) # 自动处理 .ui 文件
set(CMAKE_AUTORCC ON) # 自动处理 .qrc 资源文件

find_package(Qt6 REQUIRED COMPONENTS Core Widgets)
add_executable(app 
    main.cpp 
    LoginDialog.cpp 
    LoginDialog.hpp
    RegisterDialog.cpp 
    RegisterDialog.hpp
)

target_link_libraries(app PRIVATE Qt6::Widgets)
```
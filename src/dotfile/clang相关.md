注：在cmake项目中记得打开索引数据库：
```cmake
project(...)
# 紧接着project设置
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```
# .clang-format

```
Language: Cpp
BasedOnStyle: Google
ColumnLimit: 120

# 对齐连续的行尾注释
AlignTrailingComments: true

# 对齐连续的变量声明
# 例如:
# int a      = 1;
# int bbb    = 2;
# double ccc = 3.0;
# 取 true 会进行对齐
AlignConsecutiveDeclarations: false

# 是否允许将短函数合并到一行
# None: 绝不允许任何函数放在一行。即使是空函数也必须换行
# Empty: 只允许空函数放在一行
# Inline: 只允许空函数和 inline 修饰的短函数放在一行
# All: 允许所有符合列宽限制的短函数放在一行
AllowShortFunctionsOnASingleLine: Empty

# 不允许将简短的if语句放在一行
# Never: 绝不允许任何 if 语句放在一行
# WithoutElse: 只允许没有 else 分支的短 if 语句放在一行
AllowShortIfStatementsOnASingleLine: Never
```

```bash
# 格式化单个文件
clang-format -i main.cpp

# 格式化多个文件
clang-format -i my_class.cpp my_class.h utils/helpers.cpp

# 找到所有 .h, .cpp, .hpp, .cc 文件并用 clang-format 格式化
find . -name "*.h" -o -name "*.cpp" -o -name "*.hpp" -o -name "*.cc" | xargs clang-format -i
```

# clang配置文件

位置：`~/.config/clangd/config.yaml`

```yaml
# C++ 文件
If:
  PathMatch: '.*\.(cpp|cxx|cc|hpp|hxx|hh)$'
CompileFlags:
  Add: [-std=c++26, -Wall, -Wextra]

---
# C 文件
If:
  PathMatch: '.*\.(c|h)$'
CompileFlags:
  Add: [-std=c17, -Wall, -Wextra]

---
Diagnostics:
  # 缺少 include 的诊断（如需要）
  MissingIncludes: Strict
  # 未使用 include 的诊断:None Strict
  UnusedIncludes: None

---
# 内联提示
InlayHints:
  ParameterNames: true
  DeducedTypes: true
```

# .clang-tidy

假设正在分析的文件是 `/home/user/my_project/src/main.cpp`,Clang-tidy 会先在 `/home/user/my_project/src/` 目录寻找 `.clang-tidy`。如果没找到，它会去上一级目录 `/home/user/my_project/` 寻找。如果还没找到，它会继续去 `/home/user/` (也就是您的主目录) 寻找。如果还没找到，它会去 `/home/` 寻找。最后会找到根目录 `/`。

一旦找到，就停止向上寻找。

示例：
```yaml
# .clang-tidy
Checks: >
  -*, 
  clang-analyzer-*, 
  modernize-*, 
  google-*, 
  readability-*, 
  bugprone-*, 
  performance-*, 
  cppcoreguidelines-*, 
  cert-*

# WarningsAsErrors: '*'  # 将警告视为错误，强制修复
# 默认情况下，Clang-Tidy 通常只检查你所编译的源文件（.cpp），而不会检查它们 #include 的头文件中的问题
HeaderFilterRegex: '.*' # 检查所有头文件

CheckOptions:
  # 全局选项
  - key: readability-function-size.IgnoreMacros
    value: true          # 函数大小检查应忽略宏，减少误报

  # 软件工程与安全最佳实践
  - key: readability-braces-around-statements
    value: true          # 强制 if/for/while 的单行语句也使用花括号
  - key: google-readability-casting
    value: true          # 禁止使用 C 风格的强制类型转换
  - key: bugprone-macro-parentheses
    value: true          # 强制宏的参数和表达式被括号包围
  - key: google-explicit-constructor
    value: true          # 单参数构造函数应为 explicit，防止隐式转换
  - key: modernize-use-noexcept
    value: true          # 对可标记为 noexcept 的函数提出建议
  - key: performance-noexcept-move-constructor
    value: true          # 移动构造函数应尽可能标记为 noexcept，以优化容器操作
  - key: cppcoreguidelines-pro-type-member-init
    value: true          # 强制初始化所有成员变量，避免未定义行为

  # 可读性与维护性
  - key: readability-function-size.StatementThreshold
    value: 50            # 函数语句数上限
  - key: readability-function-size.ParameterThreshold
    value: 5             # 函数参数数量上限
  - key: readability-function-size.BranchThreshold
    value: 15            # 函数圈复杂度上限
  - key: readability-function-size.NestingThreshold
    value: 4             # 最大嵌套深度

  # 命名规范
  - key: readability-identifier-naming.NamespaceCase
    value: lower_case    # namespace 声明的命名风格 namespace my_project
  - key: readability-identifier-naming.ClassCase
    value: CamelCase     # class 命名风格：大驼峰
  - key: readability-identifier-naming.StructCase
    value: CamelCase     # struct 命名风格：大驼峰
  - key: readability-identifier-naming.EnumCase
    value: CamelCase     # enum 和 enum class 的命名风格:大驼峰
  - key: readability-identifier-naming.EnumConstantCase
    value: kCamelCase    # 枚举成员的命名风格：以k开头的大驼峰(k是Konstant(德语，意为constant))
  - key: readability-identifier-naming.FunctionCase
    value: camelBack     # 控制独立函数（非成员函数）的命名风格：小驼峰
  - key: readability-identifier-naming.MemberCase
    value: camelBack_    # 控制类的成员变量的命名风格：变量名后面加一个下划线，变量名是小驼峰
                         # 其他选项还有m_camelBack，变量名前面加个前缀m_
  - key: readability-identifier-naming.ParameterCase
    value: camelBack     # 控制函数参数的命名风格
  - key: readability-identifier-naming.VariableCase
    value: camelBack     # 控制局部变量的命名风格
  - key: readability-identifier-naming.ConstantCase
    value: kCamelCase    # 控制 const 或 constexpr 修饰的全局或静态变量的命名风格：kCamelCase
  - key: readability-identifier-naming.MacroDefinitionCase
    value: UPPER_CASE    # 控制 #define 定义的宏的命名风格，例如：#define MAX_BUFFER_SIZE 1024
  - key: readability-identifier-naming.TypeAliasCase
    value: CamelCase     # 控制类型别名 (using 和 typedef) 的命名风格
```
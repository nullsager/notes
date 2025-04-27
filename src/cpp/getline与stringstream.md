#cpp

# getline函数

**函数签名**：`istream& getline(istream& is, string& str, char delim);`

**参数解释**：

`is`：输入流对象（通常是 `cin` 或 `stringstream` 等）

`str`：存储读取结果的字符串对象。

`delim`：指定分隔符（默认是换行符 `'\n'`，但可以指定其他字符作为分隔符）。

**用法**：

`getline` 从 `is` 输入流中读取字符，直到遇到 `delim` 指定的分隔符或输入流结束。读取的内容会存储到 `str` 中，但不会包括分隔符。

**示例**：

```cpp
stringstream ss("apple,banana,orange");
string item;
// 读取直到遇到逗号
while (getline(ss, item, ',')) cout << item << endl;
```

参考资料与注意事项：[std::string 介绍](https://www.learncpp.com/cpp-tutorial/introduction-to-stdstring/)
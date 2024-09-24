# 文件操作
## fopen 函数
函数原型：`FILE *fopen(const char *filename, const char *mode);`

参数说明：

`filename`：要打开的文件的路径，可以是相对路径或绝对路径。

`mode`：指定文件的打开模式，决定了文件是以何种方式打开。常见的模式包括：
+ `r`：以只读模式打开文件。文件必须存在
+ `w`：以写入模式打开文件。如果文件存在，其内容会被截断（清空）。如果文件不存在，则会创建一个新文件
+ `a`：以追加模式打开文件。写入的数据会被添加到文件末尾。如果文件不存在，则会创建一个新文件
+ `r+`：以读写模式打开文件。文件必须存在
+ `w+`：以读写模式打开文件。如果文件存在，其内容会被截断；如果文件不存在，则会创建一个新文件
+ `a+`：以读写追加模式打开文件。写入的数据会被添加到文件末尾。如果文件不存在，则会创建一个新文件

另外，还可以在模式字符串中加入 `b` 来以二进制模式打开文件，如 `rb`, `wb`, `ab` 等，这在处理二进制文件（如图片、音频等）时非常有用

返回值:

成功时，`fopen` 返回一个指向 `FILE` 对象的指针，该指针可用于后续的文件操作。
失败时，返回 `NULL`，并设置 `errno` 以指示错误类型。
```cpp
int main() {
  // 打开文件以写入模式，如果文件不存在则创建
  FILE *file = fopen("example.txt", "w");
  if (file == NULL) {
    perror("无法打开文件");
    return 1;
  }
  // 向文件写入字符串
  const char *text = "Hello, fopen!\n";
  if (fputs(text, file) == EOF) {
    perror("写入文件失败");
    fclose(file);
    return 1;
  }
  // 关闭文件
  if (fclose(file) != 0) {
    perror("关闭文件失败");
    return 1;
  }
}
```
## fgets 函数
`fgets`用于从文件中读取一行文本。它常用于逐行读取文件内容，适用于处理文本文件

函数原型:`char *fgets(char *str, int n, FILE *stream);`

参数说明:

`str`：指向一个字符数组的指针，用于存储读取到的字符串。

`n`：要读取的最大字符数（包括终止符 `\0`）。`fgets` 会读取最多 `n-1` 个字符，确保字符串以 `\0` 结尾。

`stream`：文件指针，表示要读取的文件。

返回值: 成功时，返回 `str` 指针，即指向存储数据的缓冲区。失败或到达文件末尾时，返回 `NULL`。

工作机制
`fgets` 从指定的文件流中读取字符，直到满足以下条件之一：
+ 读取了 `n-1` 个字符。
+ 遇到了换行符 `\n`。
+ 到达文件末尾（EOF）。

读取的字符串会包括换行符（如果读取到）并以 `\0` 结束。
```cpp
int main() {
  FILE* file = fopen("example.txt", "r");
  if (!file) {
    perror("无法打开文件");
    return 1;
  }
  std::vector<char> buffer(1024); // 创建一个大小为 1024 的缓冲区
  while (fgets(buffer.data(), buffer.size(), file)) {
    std::cout << buffer.data();
  }
  if (ferror(file)) {
    perror("读取文件时出错");
  }
  fclose(file);
}
```
`data()` 是 C++11 引入的 std::vector 类的一个成员函数，用于获取指向向量内部数据的指针。对于 `std::vector<char>` 来说，它返回一个指向字符数组的 `char*` 指针。

while 里面的条件也可以改写为`fgets(buffer.empty() ? nullptr : &buffer[0], buffer.size(), filePtr.get())`
## fputs 函数
`fputs` 用于将字符串写入到指定的文件流中。它常用于将文本数据写入文件，适用于处理文本文件的写操作

函数原型: `int fputs(const char *str, FILE *stream);`

参数说明

`str`：指向一个以空字符 `\0` 结尾的字符串，该字符串将被写入到文件中。

`stream`：指向一个 `FILE` 对象，表示目标文件流。通常由 `fopen` 函数返回。

返回值

成功：返回一个非负整数。

失败：返回 `EOF`（通常为 -1），并设置 `errno` 以指示错误类型。

工作机制:

`fputs` 将字符串 `str` 中的字符（不包括终止的空字符 `\0`）写入到指定的文件流 stream 中。写入的内容会按照文件流的打开模式（如 `w`、`a` 等）进行处理

## ferror 函数
`ferror` 用于检查与给定文件流相关的错误标志，通常在文件操作（如 fread、fgets 等）完成后调用 ferror，以检查是否有错误发生

函数原型：`int ferror(FILE *stream);`

返回值:

非零值：表示在文件操作过程中发生了错误。

零：表示没有检测到错误。
## perror 函数

perror 函数用于根据当前的 errno 值输出一个描述错误信息的字符串

函数原型：`void perror(const char *s);`
```cpp
int main() {
  FILE* file = fopen("nonexistent.txt", "r");
  if (!file) {
    perror("打开文件失败");
    return 1;
  }
}
```
输出示例（假设文件不存在）：`打开文件失败: No such file or directory`

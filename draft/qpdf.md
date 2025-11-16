好的，这里为您详细讲解一下 qpdf 的使用方法。

**qpdf** 是一款功能强大的开源命令行工具，用于对 PDF 文件进行各种操作和转换。它被广泛认为是处理 PDF 结构和内容的瑞士军刀。由于其命令行特性，它非常适合集成到自动化脚本和工作流程中。

### **核心概念**

qpdf 的工作原理是理解 PDF 文件的内部结构。它可以读取一个 PDF 文件，对其内部对象进行一系列转换，然后将结果写入一个新的 PDF 文件。原始文件通常保持不变，除非您明确指定覆盖它。

### **安装 qpdf**

在开始使用之前，您需要先安装它。

  * **Windows:**

      * 可以通过 [Scoop](https://scoop.sh/) 包管理器轻松安装：`scoop install qpdf`
      * 也可以从 qpdf 的 [GitHub Releases 页面](https://github.com/qpdf/qpdf/releases)下载最新的 `.msi` 安装包进行安装。

  * **macOS:**

      * 使用 [Homebrew](https://brew.sh/) 是最简单的方式：`brew install qpdf`

  * **Linux (Debian/Ubuntu):**

      * 使用 apt 包管理器：`sudo apt-get update && sudo apt-get install qpdf`

  * **Linux (Fedora/CentOS/RHEL):**

      * 使用 dnf 或 yum：`sudo dnf install qpdf` 或 `sudo yum install qpdf`

安装完成后，可以在终端或命令提示符中输入 `qpdf --version` 来验证是否安装成功。

### **基本语法**

qpdf 的基本命令格式如下：

```bash
qpdf [options] <infilename> [<outfilename>]
```

  * `[options]`: 用于指定要执行的操作，例如合并、拆分、加密等。
  * `<infilename>`: 输入的 PDF 文件名。
  * `[<outfilename>]]`: 输出的 PDF 文件名。如果省略，qpdf 通常会将信息打印到标准输出，或者在某些操作中会报错（因为它需要一个输出文件）。

-----

### **常用功能及示例**

下面我们将通过具体的示例来讲解 qpdf 的核心功能。

#### **1. 合并多个 PDF 文件**

这是 qpdf 最常用的功能之一。可以将多个 PDF 文件合并成一个。

**语法:**

```bash
qpdf --empty --pages <file1.pdf> <pages1> <file2.pdf> <pages2> ... -- <output.pdf>
```

  * `--empty`: 创建一个空的 PDF 文件作为操作的起点。
  * `--pages`: 告诉 qpdf 后面的参数是输入文件和要包含的页面范围。
  * `<file.pdf> <pages>`: 指定要从哪个文件的哪些页面进行合并。页面范围可以是 `1-5`（第1到5页），`z`（最后一页），`1-z`（所有页面）。如果只写文件名不写页面范围，则默认包含所有页面。
  * `--`: 表示选项结束，后面的参数是输出文件名。

**示例:**

  * **合并 file1.pdf 和 file2.pdf 的所有页面：**

    ```bash
    qpdf --empty --pages file1.pdf 1-z file2.pdf 1-z -- merged.pdf
    ```

    (更简洁的写法，省略页面范围默认为全部)

    ```bash
    qpdf --empty --pages file1.pdf file2.pdf -- merged.pdf
    ```

  * **合并 file1.pdf 的第1页和 file2.pdf 的第 2-5 页：**

    ```bash
    qpdf --empty --pages file1.pdf 1 file2.pdf 2-5 -- partial_merged.pdf
    ```

#### **2. 拆分 PDF 文件**

将一个多页的 PDF 文件拆分成多个单页或多页的文件。

**语法:**

```bash
qpdf <in.pdf> --split-pages <output_prefix_%d.pdf>
```

或者

```bash
qpdf <in.pdf> --split-pages -- <output_prefix_%d.pdf>
```

  * `--split-pages`: 执行页面拆分操作。
  * `output_prefix_%d.pdf`: 输出文件的命名模板。`%d` 会被替换成页码（例如 `out_1.pdf`, `out_2.pdf`）。您也可以使用 `%02d`, `%03d` 等来补零（例如 `out_01.pdf`, `out_02.pdf`）。

**示例:**

  * **将 `document.pdf` 拆分成 `page_1.pdf`, `page_2.pdf` ...**
    ```bash
    qpdf document.pdf --split-pages -- page_%d.pdf
    ```

#### **3. 提取特定页面**

从一个 PDF 文件中提取指定的页面，生成一个新的 PDF 文件。

**语法:**
这本质上是“合并”操作的一种特殊形式，只是输入文件只有一个。

```bash
qpdf <in.pdf> --pages <in.pdf> <pages_to_extract> -- <out.pdf>
```

**示例:**

  * **从 `report.pdf` 中提取第 3、5、7-9 页：**

    ```bash
    qpdf report.pdf --pages . 3,5,7-9 -- extracted_pages.pdf
    ```

    这里的 `.` 是一个简写，代表输入文件本身。

  * **提取 `report.pdf` 的最后5页：**

    ```bash
    qpdf report.pdf --pages . -- --last-5-pages.pdf # 这是一种错误的语法，正确的方式如下
    # 需要先知道总页数，或者使用 z
    qpdf report.pdf --pages . z-4..z -- last-5-pages.pdf # z-4, z-3, z-2, z-1, z
    # 或者反向计数
    qpdf report.pdf --pages . r5-r1 -- last-5-pages.pdf # r1代表最后一页，r5代表倒数第五页
    ```

#### **4. 旋转页面**

可以对 PDF 的页面进行旋转。

**语法:**

```bash
qpdf <in.pdf> --rotate=<angle> <pages> -- <out.pdf>
```

  * `--rotate=<angle>`: 指定旋转角度。角度必须是 90 的倍数（`+90`, `-90`, `180`）。`+90` 表示顺时针旋转90度。
  * `<pages>`: 指定要旋转的页面范围。

**示例:**

  * **将 `scanned.pdf` 的所有奇数页顺时针旋转 90 度：**
    ```bash
    qpdf scanned.pdf --rotate=+90 1-z:odd -- rotated_odd.pdf
    ```
  * **将 `document.pdf` 的第 1 页和第 3 页旋转 180 度：**
    ```bash
    qpdf document.pdf --rotate=180 1,3 -- rotated_specific.pdf
    ```

#### **5. 加密和解密 PDF**

qpdf 可以为 PDF 添加密码保护，也可以移除密码（需要知道密码）。

**语法 (加密):**

```bash
qpdf <in.pdf> --encrypt <user-password> <owner-password> <key-length> [ -- <permission> ... ] -- <out.pdf>
```

  * `<user-password>`: 打开文件时需要输入的用户密码。
  * `<owner-password>`: 拥有所有权限的所有者密码。
  * `<key-length>`: 加密密钥长度，可以是 `40`, `128`, 或 `256`。推荐使用 `256`。
  * `[ -- <permission> ... ]`: 限制权限，例如 `--print=none` (禁止打印), `--modify=none` (禁止修改)。

**示例 (加密):**

  * **给 `confidential.pdf` 添加密码，用户密码是 `1234`, 所有者密码是 `abcd`，允许打印但不允许修改：**
    ```bash
    qpdf confidential.pdf --encrypt 1234 abcd 256 --print=full --modify=none -- encrypted.pdf
    ```

**语法 (解密):**

```bash
qpdf --decrypt --password=<password> <in.pdf> <out.pdf>
```

**示例 (解密):**

  * **移除 `encrypted.pdf` 的密码保护，密码是 `1234`：**
    ```bash
    qpdf --decrypt --password=1234 encrypted.pdf decrypted.pdf
    ```

#### **6. Web 优化 (线性化)**

线性化的 PDF 文件可以更快地在网页上显示，因为它允许浏览器先下载和显示第一页，而不是等待整个文件下载完成。

**语法:**

```bash
qpdf <in.pdf> --linearize <out.pdf>
```

**示例:**

  * **对 `large_file.pdf` 进行 Web 优化：**
    ```bash
    qpdf large_file.pdf --linearize web_optimized.pdf
    ```

#### **7. 检查和修复 PDF**

qpdf 可以用来检查 PDF 文件是否存在语法问题，并在某种程度上进行修复。它在写入新文件时，会自动重建文件的交叉引用表（xref table），这个过程本身就能修复很多常见的 PDF 损坏问题。

**语法:**

```bash
qpdf --check <in.pdf>
```

如果只想修复文件，可以简单地将其“复制”一遍：

```bash
qpdf <in.pdf> <repaired.pdf>
```

**示例:**

  * **检查 `corrupted.pdf` 文件：**

    ```bash
    qpdf --check corrupted.pdf
    ```

    (该命令会把检查信息输出到控制台)

  * **尝试修复 `corrupted.pdf`：**

    ```bash
    qpdf corrupted.pdf repaired.pdf
    ```

#### **8. 高级功能：查看内部结构**

qpdf 可以将 PDF 的内部结构导出为 JSON 或 QDF 格式，这对于开发者和需要深入分析 PDF 的用户非常有用。

  * **导出为 QDF (qpdf 数据格式):**

    ```bash
    qpdf <in.pdf> --qdf <out.qdf>
    ```

    这个 QDF 文件是文本格式的，可读性强，可以手动编辑。编辑后，可以再转换回 PDF：

    ```bash
    qpdf <edited.qdf> <new.pdf>
    ```

    **警告:** 直接编辑 QDF 文件需要对 PDF 内部结构有深入的了解，否则很容易生成无效的 PDF。

  * **导出为 JSON:**

    ```bash
    qpdf --json <in.pdf> <out.json>
    ```

    这会将 PDF 的结构以 JSON 格式输出，方便程序进行解析和处理。

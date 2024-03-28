+++
title = "使用 OpenPyXL 读写 excel 大文件"
date = "2023-05-07T19:31:45+08:00"
categories = [ "编程",]
tags = [ "code", "python",]
toc = "true"
+++


使用 python OpenPyXL 读写 excel 大文件时，有专门的 read_only write-only 模式来提升读写效率。

<!--more-->

### openpyxl read_only mode

```python
from openpyxl import load_workbook
wb = load_workbook(filename='large_file.xlsx', read_only=True)
ws = wb['big_data']

# min_col&max_col: 只处理 B 列，min_row=2: 从第二行开始，只读取值
col_index_B = openpyxl.utils.column_index_from_string('B') 
for cell_value in ws.iter_rows(min_row=2, min_col=col_index_B, max_col=col_index_B, values_only=True):
    print(cell_value)

# Close the workbook after reading
wb.close()
```
#### 只读取第一行的错误
如果你的 excel 文件是通过第三方软件 (数据库客户端) 或者代码生成的，很容易遇到一个问题就是上面的`ws.iter_rows`或`ws.rows`遍历只会读取第一行。这是因为 read only 模式在 load_workbook 时只读取了文件的元信息，在遍历时也依赖 worksheet 的元信息，很多非 office 生成的 excel 没有正确设置元信息。你可以通过`ws.calculate_dimension()`检查 excel 行列信息，如果返回的是`A1:A1`等与实际大小不一致的情况，可以通过`ws.reset_dimensions()`来重置`ws`的`max_row` and `max_column`属性。
注意，`ws.reset_dimensions()`会读取整个文件，效率会降低到非 read_only 模式一样。

哎，这个坑花了一下午的排查，就是不认真看一下[文档](https://openpyxl.readthedocs.io/en/latest/optimized.html#worksheet-dimensions)

### openpyxl write_only mode
1. 确保安装了`lxml` `openpyxl`两个库
2. 使用 write_only 写大数据
```python
from openpyxl import Workbook
wb = Workbook(write_only=True)
ws = wb.create_sheet()

# now we'll fill it with 100 rows x 200 columns
for irow in range(100):
    ws.append(['%d' % i for i in range(200)])
   
# save the file
wb.save('new_big_file.xlsx') # doctest: +SKIP
```
### 官方文档
[openpyxl Optimised Modes](https://openpyxl.readthedocs.io/en/latest/optimized.html)

### openpyxl vs xlsxwriter
1. 两个常用功能和性能上差别不大，部分样式设置有差别。
2. pandas 支持两个，可以通过 engine 参数切换。

### 使用 polars
polars 是使用 rust 语言编写的 DataFrame 库，基于 arrow 格式，提供 rust、python、nodejs 三种编程语言接口。目前 polars 支持读取整个 excel 文件，但是写文件不行，只能通过转换成 pandas 的 df 再处理。本地测试一下 polars 读取整个 excel 文件，速度大约是 pandas 普通模式的两倍，内存使用是 pandas 的 2/3，这个性能和内存，感觉在 python 领域希望不大，看 rust 那边的发展了。

polars 的 API 一般是尽可能和 pandas 保持一致，所以使用起来也比较简单，遇到 API 缺失的，可以直接转换成 pandas 的 df 使用`df.to_pandas()`

```python
import pandas as pd
import polars as pl  # read_excel 还需要 xlsx2csv lib

df = pl.read_excel(
    "test.xlsx",
    sheet_id=1,
    xlsx2csv_options={"skip_empty_lines": True},
    read_csv_options={"has_header": True},
)
print(f'{type(df)=}')  # class 'polars.dataframe.frame.DataFrame'
print(f'{df.shape=}')
print(f'{df.head()=}')
for cell in df['Courses']:
    print(f'{cell=}')
```
polars 的输出格式比 pandas 的好看多了，使用了 box-drawing 相关的 unicode 字符打印表格，不过需要编程字体支持才能对齐。
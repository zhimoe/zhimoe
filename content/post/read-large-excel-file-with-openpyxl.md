---
title: "使用OpenPyXL read_only write-only模式读写excel大文件"
date: "2023-05-07T19:31:45+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - python
---

使用python OpenPyXL读写excel大文件时，有专门的read_only write-only模式来提升读写效率。

<!--more-->

### openpyxl read_only mode

```python
from openpyxl import load_workbook
wb = load_workbook(filename='large_file.xlsx', read_only=True)
ws = wb['big_data']

# min_col&max_col: 只处理B列, min_row=2: 从第二行开始, 只读取值
col_index_B = openpyxl.utils.column_index_from_string('B') 
for cell_value in ws.iter_rows(min_row=2, min_col=col_index_B, max_col=col_index_B, values_only=True):
    print(cell_value)

# Close the workbook after reading
wb.close()
```
#### 只读取第一行的错误
如果你的excel文件是通过第三方软件(数据库客户端)或者代码生成的，很容易遇到一个问题就是上面的`ws.iter_rows`或`ws.rows`遍历只会读取第一行。 这是因为read only模式在load_workbook时只读取了文件的元信息，在遍历时也依赖worksheet的元信息，很多非office生成的excel没有正确设置元信息。 你可以通过`ws.calculate_dimension()`检查excel行列信息，如果返回的是`A1:A1`等与实际大小不一致的情况，可以通过`ws.reset_dimensions()`来重置`ws`的`max_row` and `max_column`属性。
注意，`ws.reset_dimensions()`会读取整个文件，效率会降低到非read_only模式一样。

哎，这个坑花了一下午的排查，就是不认真看一下[文档](https://openpyxl.readthedocs.io/en/latest/optimized.html#worksheet-dimensions)

### openpyxl write_only mode
1. 确保安装了`lxml` `openpyxl`两个库
2. 使用write_only写大数据
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
2. pandas支持两个，可以通过engine参数切换。

### 使用polars
polars是使用rust语言编写的DataFrame库，基于arrow格式，提供rust、python、nodejs三种编程语言接口。 目前polars支持读取整个excel文件，但是写文件不行，只能通过转换成pandas的df再处理。本地测试一下polars读取整个excel文件，速度大约是pandas普通模式的两倍，内存使用是pandas的2/3，这个性能和内存，感觉在python领域希望不大，看rust那边的发展了。

polars的API一般是尽可能和pandas保持一致，所以使用起来也比较简单，遇到API缺失的，可以直接转换成pandas的df使用`df.to_pandas()`

```python
import pandas as pd
import polars as pl  # read_excel还需要xlsx2csv lib

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
polars的输出格式比pandas的好看多了，使用了box-drawing相关的unicode字符打印表格，不过需要编程字体支持才能对齐。
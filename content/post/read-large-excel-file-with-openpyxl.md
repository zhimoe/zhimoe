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

for row in ws.rows:
    for cell in row:
        print(cell.value)

# Close the workbook after reading
wb.close()
```
#### 只读取第一行的错误
如果你的excel文件是通过第三方软件(数据库客户端)或者代码生成的，很容易遇到一个问题就是上面的`ws.rows`遍历只会读取第一行。 这是因为read only模式在load_workbook时只读取了文件的元信息，在遍历时也依赖worksheet的元信息，很多非office生成的excel没有正确设置元信息。 你可以通过`ws.calculate_dimension()`检查excel行列信息，如果返回的是`A1:A1`等与实际大小不一致的情况，可以通过`ws.reset_dimensions()`来重置`ws`的`max_row` and `max_column`属性。

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

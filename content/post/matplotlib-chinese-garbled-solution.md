---
title: "Matplotlib图例中文乱码解决方案"
date: "2020-05-01T19:18:05+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - python
---
很久以前写的一个答案,四年来一直有人评论感谢,说只有我的方法是有效的.非常意外也很高兴. 也放到博客中里备份吧.
<!--more-->
[zhihu.com](https://www.zhihu.com/question/25404709/answer/67672003)

```python
# https://www.zhihu.com/question/25404709/answer/67672003
import matplotlib.font_manager as fm
# 微软雅黑,如果需要宋体,可以用simsun.ttc
myfont = fm.FontProperties(fname='C:/Windows/Fonts/msyh.ttc')
# Linux字体在"/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc", 
# 需要先安装字体">sudo apt install fonts-noto-cjk -y"
# MacOS中文字体文件在"/System/Library/Fonts/PingFang.ttc"
# Win10,Linux已测试,MacOS未验证

import matplotlib.pyplot as plt

plt.clf()  # 清空画布
plt.plot([1, 2, 3], [4, 5, 6])
plt.xlabel("横轴",fontproperties=myfont)
plt.ylabel("纵轴",fontproperties=myfont)
plt.title("pythoner.com",fontproperties=myfont)
plt.legend(['图例'],prop=myfont)
plt.show()
```

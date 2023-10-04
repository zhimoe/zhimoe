+++
title = "Jupyter Notebook Install New Package[翻译]"
date = "2018-11-01T19:04:32+08:00"
categories = [ "翻译",]
tags = [ "code", "python",]
toc = "true"
+++


### notes
在pycharm写代码中如果使用到新的package,例如`numpy`,只需要输入`import numpy` 然后ALT+ENTER在提示中选择install new package即可.

在notebook中,网上的教程都说是`!pip install numpy`. 但是这个可能有坑.究其原因是因为:

<!--more-->


通过bash启动的notebook中的`python pip conda`这几个命令的环境和实际执行notebook代码的python环境可能不是同一个.
这种情况一般发生在系统有好几个python的情况,例如系统自带python和用户安装的anaconda python.
可以通过对比以下两个notebook命令的输出判断pip执行环境和notebook代码执行环境是否一致：
```python
# pip执行环境python
!type python
# notebook 代码执行环境的python
import sys
sys.executable
```
如果不一样,那么需要使用下面命令安装才能在notebook中生效：
import sys
!{sys.executable} -m pip install numpy


### sys和os区别

```text
os: 操作系统的抽象.

sys: 代码和python解释器交互的接口.提供一系列函数来访问修改python解释器环境设置.
```

source:[installing new python package from jupyter notebook](https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/)

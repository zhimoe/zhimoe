+++
title = "Jupyter Notebook Install New Package[翻译]"
date = "2018-11-01T19:04:32+08:00"
categories = [ "翻译",]
tags = [ "code", "python",]
toc = "true"
+++


### notes
在 pycharm 写代码中如果使用到新的 package，例如`numpy`,只需要输入`import numpy` 然后 ALT+ENTER 在提示中选择 install new package 即可。

在 notebook 中，网上的教程都说是`!pip install numpy`. 但是这个可能有坑。究其原因是因为：

<!--more-->


通过 bash 启动的 notebook 中的`python pip conda`这几个命令的环境和实际执行 notebook 代码的 python 环境可能不是同一个。
这种情况一般发生在系统有好几个 python 的情况，例如系统自带 python 和用户安装的 anaconda python.
可以通过对比以下两个 notebook 命令的输出判断 pip 执行环境和 notebook 代码执行环境是否一致：
```python
# pip 执行环境 python
!type python
# notebook 代码执行环境的 python
import sys
sys.executable
```
如果不一样，那么需要使用下面命令安装才能在 notebook 中生效：
import sys
!{sys.executable} -m pip install numpy


### sys 和 os 区别

```text
os: 操作系统的抽象。

sys: 代码和 python 解释器交互的接口。提供一系列函数来访问修改 python 解释器环境设置。
```

source:[installing new python package from jupyter notebook](https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/)

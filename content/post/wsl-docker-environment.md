+++
title = "wsl-docker-environment"
date = 2019-01-01
categories = [ "编程",]
tags = [ "wsl", "code",]
toc = "true"
+++


使用 wsl,MobaXterm,cmder,docker 打造可视化的 linux 开发环境

离不开 Windows 的理由很多，作为后端开发需要使用 linux 的情况也很多，双系统总归是不方便，而且 linux 下的 GUI 体验也没用 Win 10 好。
如果使用虚拟机，那么文件交换和网络等各种问题也需要解决，对系统的内存要求也更高一些。微软为了让更多的开发人员留在 Win10 上面，开发了 WSL，目前的实际体验已经很棒，
今天介绍一下如何打造一个可视化的 linux 开发环境--即在 Win10 启动 linux 的 GUI 软件，例如 vs code 等。在 wsl 启动 vs code 写代码可以有效避免一些 Windows 和 linux 的编码和换行问题。

本教程分为 2 部分：

- [x] 配置 wsl 可视化
- [x] 在 wsl 使用 docker

> 以下内容中 `wsl`和`ubuntu`含义相同，`console`和`命令行`含义相同。

<!--more-->

## 配置 wsl 可视化
系统要求是 Win 10 1803+ 版本 (低于 1803 的 wsl 功能有问题),必须是专业版或教育版才有 wsl 功能。以下内容的命令行如果开头有`>`字符请忽略。

### windows 开启 wsl 功能
控制面板\程序\程序和功能\开发或关闭 Windows 功能 > 勾选 '适用于 linux 的 Windows 子系统' 和 'hyper-V'(docker for Windows 需要这个功能，也可以使用 virtualbox 代替), 重启电脑。

### windows 下载 wsl
Windows store 搜索"wsl"或者"ubuntu"下载 ubuntu 版本。ubuntu 和 ubuntu1804 是一个版本，ubuntu1604 是旧的版本。安装完成你的 Windows 应用列表会有一个 ubuntu 应用，点击图标即可打开 ubuntu 命令行。第一次启动需要等待初始化，然后设置用户名和密码。由于字体难看，所以不用这个自带的命令行而使用下面的 cmder.

### windws 下载 cmder 软件
cmder 是 Windows 下最强的命令行功能。不要下载 mini 那个，里面没用 vim 和 git.第一次启动 cmder 记得修改 cmder 启动目录 (默认是 c 盘) 和显示中文设置，具体方法请 google.

### wsl 修改软件源，使用阿里云的源。
```bash
> sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
> sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
> sudo apt update
> sudo apt upgrade -y
```

### wsl 安装必要软件
```bash
# 安装你需要的软件,git和vim是必须的,后面的编辑命令是使用vim
>sudo apt install openjdk-8-jdk-headless openjdk-8-jre-headless maven git unzip vim -y
```

### 修改 wsl 下 Windows 磁盘挂载点
默认的 Windows 磁盘在 wsl 的访问方式是`/mnt/d/`开头，d 表示 d 盘。但是 docker on linux 的访问路径是 `/d`,所以这里需要修改挂载点路径。

```bash
sudo vim /etc/wsl.conf
##添加3行内容
[automount] 
root = / 
options = "metadata"
```
退出 wsl 重启，发现`/mnt`已经没了，当前目录应该是`/c/xxx`或者`/d/xxx`.

### wsl 安装 vs code 和中文字体
因为 wsl 没用中文字体将显示豆腐块。

```bash
# install chinese fonts for wsl,font name:  'Noto Sans Mono CJK SC'
sudo apt install -y fonts-noto-cjk fonts-noto-cjk-extra
# Win10下载vs code的deb包,cd到该目录,使用下面命令安装
sudo apt install ./code_1.31.1-1549938243_amd64.deb
# 在wsl要启动code必要依赖
sudo apt install libgtk2.0-0 libxss1 libasound2
```

### wsl 设置 SSH 功能
这样可以借助 VcXsrv 的 X11 转发功能打开 GUI 软件

```bash
>sudo vim /etc/ssh/sshd_config
#取消Port的注释,并将端口改为2222 (端口需要大于1000)
#将PasswordAuthentication的值改为yes.

#重启 ssh server:
sudo service ssh --full-restart

#将ssh server设置为服务:
sudo service ssh start
```

### windows 安装 VcXsrv
用它的 X11 转发功能。安装后默认选项即可，可以设置为开机启动。

### 启动 wsl 的 vs code
在 wsl 输入`code .`,等待 2 秒，你会发现 Windows 任务栏启动了一个 vs code，如果没用启动成功，说明你的 VcXsrv 的 X11 转发功能有问题。

### 配置 vs code. 
上面打开的 vs code 有 2 个问题：中文显示豆腐块，和不能全屏。打开 vs code 的设置
```bash
#在字体里面先设置你想要英文字体,逗号跟上'Noto Sans Mono CJK SC'
#搜索titleBarStyle,将'Window: title Bar Style'设置为 native
#上面2个设置也可通过直接编辑文件设置,例如我的vs code文件设置是
> cat ~/.config/Code/User/settings.json
{
    "Window.titleBarStyle": "native",
    "editor.fontFamily": "monospace,'Noto Sans Mono CJK SC'"
}
```
至此，已经可以在 linux 下面开发了。当然，其他 GUI 软件没用测试不确定是不是会有小问题。但是 vs code 已经可以应付很多开发工作了。

## 在 wsl 使用 docker

目前的 wsl 是不支持运行 docker 的，但是可以在 wsl 使用 Windows 的 docker，在使用上面是无感的。

- 安装 docker for Windows. 这个就不细说了，注意 docker 社区版也是需要注册才能下载的。

- 启动 docker for Windows，右键任务栏的 docker 图标，"settings",勾上 "expose the daemon on tcp:/localhost:2375 without TLS",这样在 wsl 可以访问这个 docker 服务。

- wsl 安装 docker，详细内容可以参考官方文档，下面仅列出必要 bash 命令。
  
    ```bash
    #安装必要组件
    sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
    #gpg 签名
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo apt-key fingerprint 0EBFCD88
    #添加 docker 安装源
    sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"

    sudo apt update
    sudo apt install -y docker-ce
    #通过 pip 安装 docker-compose
    sudo apt install -y python python-pip
    sudo usermod -aG docker $USER
    pip install --user docker-compose
    #验证 docker 安装是否成功
    docker info
    docker-compose --version

    #修改 docker 服务为 Windows 的 docker
    echo "export DOCKER_HOST=tcp://localhost:2375" >> ~/.bashrc && source ~/.bashrc

    #验证是否可以访问Windows的docker服务,看image list命令输出和Windows的命令行下面的image list输出是不是完全一样. 可以先在Windows下用docker拉几个镜像.然后在wsl验证
    docker image list
    ```

至此，wsl 的 docker 服务也配置完成。


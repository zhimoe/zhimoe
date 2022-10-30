---
title: "在macOS通过SSH访问Windows的WSL2 Ubuntu"
date: "2022-10-30T14:14:59+08:00"
toc: true
categories:
 - "编程"
tags:
 - ssh
 - mac
 - wsl
---
配置Windows和WSL2, 使得能通过其他电脑远程SSH到WSL2 Ubuntu. 

### 背景
之前的电脑配置是LinuxMint台式机 + M1 macbook笔记本. 使用Linux主要原因是命令行和Docker. 最近由于二十大, 工作VPN在macOS不让用, 只能将台式机安装上Win10, 发现docker在WSL2运行非常丝滑, 这样正好可以当作macbook的Docker服务器. 切换到Windows还有一个原因就是, Linux的桌面真的不行, 最近三年各种版本的桌面使用一圈, Budgie,Gnome,Cinnamon,Xfce这些桌面总是偶尔界面失去响应,KDE用的不多, 卡顿没遇到但是启动总是慢半秒. Win10除了没有Bash/Zsh, 中文字体垃圾点, 其他的都完胜Linux. 
下面的教程主要参考:[Configuring SSH access into WSL 1 and WSL 2](https://jmmv.dev/2022/02/wsl-ssh-access.html)

<!--more-->

### 1 Win10 安装 WSL2 Ubuntu
注意, 是安装WSL2. 方法参考这个[enable-virtual-machine-feature](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-3---enable-virtual-machine-feature): 

1. 以管理员身份打开 PowerShell（“开始”菜单 >“PowerShell” >单击右键 >“以管理员身份运行”）, 然后输入以下命令:
 `dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`;
2. 安装 "适用于 x64 计算机的 WSL2 Linux 内核更新包";
3. 将WSL设置默认version 2, in powershell: `wsl --set-default-version 2`;
4. 安装Ubuntu, in powershell: `wsl --install -d Ubuntu-22.04`;

[更多参考](https://learn.microsoft.com/zh-cn/windows/wsl/install) .  


### 2 配置SSH server（在Ubuntu执行）
进入Ubuntu
```sh
#修改软件源
sudo sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sudo apt update && sudo apt upgrade -y
```
安装配置ssh服务
```sh
sudo apt install openssh-server

sudo vim /etc/ssh/sshd_config
# 修改下面几个配置
# Port 2222
# AddressFamily any
# ListenAddress 0.0.0.0
# PasswordAuthentication yes

# 如果启动遇到这个错误, 请执行下面命令: sshd: no hostkeys available -- exiting
sudo ssh-keygen -A

# 启动ssh服务
sudo /usr/sbin/service ssh start
```

### 3 Win10防火墙设置
打开控制面板\系统和安全\Windows Defender 防火墙. 

- 最左边有高级设置
- 右键点击入站规则
- 新建入站规则
- 点击端口,特定端口设置2222
- 然后命名之后一路下一步就行

或者通过shell设置防火墙端口,以管理员身份打开 PowerShell:
```powershell
New-NetFirewallRule -Name sshd -DisplayName 'sshd for WSL' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 2222
```

### 4 本地验证SSH访问Ubuntu
打开Windows Terminal, 尝试ssh访问Ubuntu
```
ssh -p 2222 wsluser@localhost

```
如果连接上说明ssh配置已经完成. 

### 5 Ubuntu ssh服务开机自动启动
WSL2的Ubuntu ssh不是跟着Win10开机自动启动的. 在Win10的`%USERPROFILE%`目录下面新建文件`sshd.bat`
```
rem sshd.bat
@echo off
setlocal

C:\Windows\System32\bash.exe -c "sudo /usr/sbin/service ssh start"
rem C:\Windows\System32\wsl.exe -e "sudo /usr/sbin/service ssh start"

endlocal
```
注意上面 `bash.exe -c` 和 `wsl.exe -e` 两个功能是一样的. bash后面不维护了, wsl是官方推荐命令, 但是bash有输出. 

接下来把上面的脚本配置成开机自动执行:
- 按下Win键, 搜索“任务计划程序”, 右边点击“创建任务”,  
- 常规:设置任务名字“Start WSL SSH”, 勾选上“使用最高权限运行”（这是给后面网卡映射命令的权限）
- 触发器: 新建, 选择“启动时”
- 操作: 选择上面的sshd.bat脚本文件. 
保存, 重启电脑, 打开Terminal, 重新试试`ssh -p 2222 wsluser@localhost`

### 5 网卡映射
到目前为止, 在Win10本地已经可以在开机后直接通过SSH访问Ubuntu了, 但是你如果在局域网内的其他电脑访问, 还是连不上的. 这是因为WSL2是个虚拟机. 

> WSL 2 is a well-hidden virtual machine, but it is still a virtual machine—and the consequences of this design are leaky. The network interface we see within WSL is a virtual interface that does not match the physical interface that Windows manages. Windows does a good job at hiding this fact when operating directly on the local machine (e.g. you can SSH into WSL from localhost and it will work), but attempts to reach WSL from a separate machine will fail.

设置开机自动执行网卡映射命令, 将上面的sshd.bat文件改成如下:
```bat
rem ssh.bat
@echo off
setlocal

C:\Windows\System32\bash.exe -c "sudo /usr/sbin/service ssh start"
rem C:\Windows\System32\wsl.exe -e "sudo /usr/sbin/service ssh start"

C:\Windows\System32\netsh.exe interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0 protocol=tcp

for /f %%i in ('wsl hostname -I') do set IP=%%i
C:\Windows\System32\netsh.exe interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=2222 connectaddress=%IP%

endlocal
```
保存后启动, 在macbook试试, 成功.
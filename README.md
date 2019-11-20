**Hadoop-Environment-Configuration**
# The latest Hadoop environment configuration tutorial (constantly updated)  
-----
This `read.me` will teach you how to configure the environment of Hadoop based on CentOS7. There are also some tips that you need to know.

- 基于CentOS7平台的Hadoop安装及环境搭建全教程以及需要注意的操作和坑。</font>
>Update on 19,Nov,2019  

**Reference：**

Hadoop安装原教程：[centos安装hadoop超级详细没有之一  jimuka_liu](https://blog.csdn.net/jimuka_liu/article/details/82784313)

JAVA JDK安装教程：[Centos7中yum安装jdk及配置环境变量](https://www.cnblogs.com/52lxl-top/p/9877202.html)

笔者综合对比N个教程，在原有教程上进行了补充和优化，加入了配图以及更加详细的步骤介绍以及可能会遇到的各种坑。

-----
# 1 前期操作
## 1.1 Hadoop用户的创建
如果你安装 CentOS 的时候不是用的 “hadoop” 用户，那么需要增加一个名为 hadoop 的用户。

首先点击左上角的 “应用程序” -> “系统工具” -> “终端”，首先在终端中输入 su ，按回车，输入 root 密码以 root 用户登录，接着执行命令创建新用户 hadoop:
```shell
su              # 上述提到的以 root 用户登录
useradd -m hadoop -s /bin/bash   # 创建新用户hadoop
```
Shell 命令

这条命令创建了可以登陆的 hadoop 用户，并使用 /bin/bash 作为shell。

 

CentOS创建hadoop用户

接着使用如下命令修改密码，按提示输入两次密码，可简单的设为 “hadoop”（密码随意指定，若提示“无效的密码，过于简单”则再次输入确认就行）:

passwd hadoop
Shell 命令

可为 hadoop 用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题，执行：

visudo
Shell 命令

如下图，找到 root ALL=(ALL) ALL 这行（应该在第98行，可以先按一下键盘上的 ESC 键，然后输入 :98 (按一下冒号，接着输入98，再按回车键)，可以直接跳到第98行 ），然后在这行下面增加一行内容：hadoop ALL=(ALL) ALL （当中的间隔为tab），如下图所示：

 

为hadoop增加sudo权限

sudo是linux系统管理指令，是允许系统管理员让普通用户执行一些或者全部的root命令的一个工具

添加上一行内容后，先按一下键盘上的 ESC 键，然后输入 :wq (输入冒号还有wq，这是vi/vim编辑器的保存方法)，再按回车键保存退出就可以了。

最后注销当前用户(点击屏幕右上角的用户名，选择退出->注销)，在登陆界面使用刚创建的 hadoop 用户进行登陆。（如果已经是 hadoop 用户，且在终端中使用 su 登录了 root 用户，那么需要执行 exit 退出 root 用户状态）

使用 hadoop 用户登录后，还需要安装几个软件才能安装 Hadoop。
————————————————
版权声明：本文为CSDN博主「jimuka_liu」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/jimuka_liu/article/details/82784313

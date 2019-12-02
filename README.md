# 基于CentOS7平台的Hadoop安装及环境搭建全教程

### The latest Hadoop environment configuration tutorial (constantly updated) 
-----
This `read.me` will teach you how to configure the environment of Hadoop based on CentOS7. There are also some tips that you need to know.

- 基于CentOS7平台的Hadoop安装及环境搭建全教程以及需要注意的操作和坑。</font>
>Update on 19,Nov,2019  

**Reference：**

Hadoop安装原教程(原文链接)：[centos安装hadoop超级详细没有之一  jimuka_liu](https://blog.csdn.net/jimuka_liu/article/details/82784313)

JAVA JDK安装教程：[Centos7中yum安装jdk及配置环境变量](https://www.cnblogs.com/52lxl-top/p/9877202.html)

笔者综合对比N个教程，在原有教程上进行了补充和优化，加入了配图以及更加详细的步骤介绍以及可能会遇到的各种坑。

-----
## 1 前期操作

**在操作过程中，如果遇到权限相关的问题，基本上在代码前面加sudo就可以解决（这样可以暂时获取权限）**

### 1.1 Hadoop用户的创建
如果你安装 CentOS 的时候不是用的 “hadoop” 用户，那么需要增加一个名为 hadoop 的用户。
**如果不需要创建用户，则直接开始步骤1.2**

首先点击左上角的 “应用程序” -> “系统工具” -> “终端”，首先在终端中输入 su ，按回车，输入 root 密码以 root 用户登录，接着执行命令创建新用户 hadoop:
```shell
su              # 上述提到的以 root 用户登录
useradd -m hadoop -s /bin/bash   # 创建新用户hadoop
```
这条命令创建了可以登陆的 hadoop 用户，并使用 /bin/bash 作为shell。

CentOS创建hadoop用户

接着使用如下命令修改密码，按提示输入两次密码，可简单的设为 “hadoop”（密码随意指定，若提示“无效的密码，过于简单”则再次输入确认就行）:
```shell
passwd hadoop
```
可为 hadoop 用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题，执行：
```shell
visudo
```

找到 root ALL=(ALL) ALL 这行（应该在第98行，可以先按一下键盘上的 ESC 键，然后输入 :98 (按一下冒号，接着输入98，再按回车键)，可以直接跳到第98行 ），然后在这行下面增加一行内容：hadoop ALL=(ALL) ALL （当中的间隔为tab）

为hadoop增加sudo权限

sudo是linux系统管理指令，是允许系统管理员让普通用户执行一些或者全部的root命令的一个工具

添加上一行内容后，先按一下键盘上的 ESC 键，然后输入 :wq (输入冒号还有wq，这是vi/vim编辑器的保存方法)，再按回车键保存退出就可以了。

最后注销当前用户(点击屏幕右上角的用户名，选择退出->注销)，在登陆界面使用刚创建的 hadoop 用户进行登陆。（如果已经是 hadoop 用户，且在终端中使用 su 登录了 root 用户，那么需要执行 exit 退出 root 用户状态）

使用 hadoop 用户登录后，还需要安装几个软件才能安装 Hadoop。

**接下来检查系统是否联网，如果是联网状态，即可安装SSH和Java**

### 1.2 安装SSH，配置SSH无密码登录

集群、单节点模式都需要用到 SSH 登陆（类似于远程登陆，你可以登录某台 Linux 主机，并且在上面运行命令），一般情况下，CentOS 默认已安装了 SSH client、SSH server，打开终端执行如下命令进行检验：
```shell
rpm -qa | grep ssh
```
如果返回的结果包含了 SSH client 跟 SSH server，则不需要再安装。

若需要安装，则可以通过 yum 进行安装（安装过程中会让你输入 [y/N]，输入 y 即可）：
```shell
sudo yum install openssh-clients
sudo yum install openssh-server
```
接着执行如下命令测试一下 SSH 是否可用：
```shell
ssh localhost
```
此时会有提示(SSH首次登陆提示)，输入 yes 。然后按提示输入密码 hadoop，这样就登陆到本机了。

但这样登陆是需要每次输入密码的，我们需要配置成SSH无密码登陆比较方便。

首先输入 exit 退出刚才的 ssh，就回到了我们原先的终端窗口，然后利用 ssh-keygen 生成密钥，并将密钥加入到授权中：
```shell
exit                           # 退出刚才的 ssh localhost
cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat id_rsa.pub >> authorized_keys  # 加入授权
chmod 600 ./authorized_keys    # 修改文件权限
```
~的含义

在 Linux 系统中，~ 代表的是用户的主文件夹，即 “/home/用户名” 这个目录，如你的用户名为 hadoop，则 ~ 就代表 “/home/hadoop/”。 此外，命令中的 # 后面的文字是注释。

此时再用 ssh localhost 命令，无需输入密码就可以直接登陆了。

### 1.3 安装JAVA环境

**这里需要注意一下ubuntu和centOS的区别：对于安装操作，ubuntu中可以用的apt-get install，在centOS中不可以使用，需要用yum**

**输入yum list，可以查找到使用yum可以安装的所有东西。**

**同样地，如果需要root权限的话，可以通过sudo暂时获得权限，即：sudo yum install ……**

Java 环境可选择 Oracle 的 JDK，或是 OpenJDK，现在一般 Linux 系统默认安装的基本是 OpenJDK，如 CentOS 6.4 就默认安装了 OpenJDK 1.7。按 http://wiki.apache.org/hadoop/HadoopJavaVersions 中说的，Hadoop 在 OpenJDK 1.7 下运行是没问题的。需要注意的是，CentOS 6.4 中默认安装的只是 Java JRE，而不是 JDK，为了开发方便，我们还是需要通过 yum 进行安装 JDK。

系统版本
```shell
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
```
安装之前先查看一下有无系统自带jdk(如果有的话，安装可能会报错)
```shell
rpm -qa |grep java

rpm -qa |grep jdk

rpm -qa |grep gcj
```
如果有，就使用批量卸载命令
```shell
rpm -qa | grep java | xargs rpm -e --nodeps 
```
直接yum安装1.8.0版本openjdk
```shell
[root@localhost ~]# yum install java-1.8.0-openjdk* -y
```
查看版本
```shell
[root@localhost ~]# java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
```
默认jre jdk 安装路径是/usr/lib/jvm 下面
此处插入图片

JAVA_HOME指向一个含有java可执行程序的目录(一般是在 bin/java中,此目录为/bin/java的上级目录),用cd 命令进入到 jvm下唯一的一个目录中 java-1.8.0-openjdk-1.8.0.161-0.b14.el7_3.x86_64,发现其下目录为 /jar/bin/java.jre-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64 这个链接是指向 java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/jre 这个文件夹，所以，可以直接用export命令将 JAVA_HOME 指向
 jre-1.8.0-openjdk-1.8.0.121-0.b14.el7_4.x86_64这个链接.
 
 临时生效
 ```shell
 [root@localhost ~]#  export JAVA_HOME=/usr/lib/jvm/<span style="font-family: Arial;">jre-1.8.0-openjdk-1.8.0.121-0.b13.el7_3.x86_64</span> 
 ```
 当前用户生效的配置
 ```shell
 vim ~/.bashrc
#在文件底部加入下面一句
export  JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64
```
如果使所有用户生效的配置
```shell
vim /etc/profile
 #set java environment  
export JAVA_HOME=/usr/lib/jvm/java
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/jre/lib/rt.jar
export PATH=$PATH:$JAVA_HOME/bin
```
**这里需要注意一下Linux系统中的文本编辑的几个工具vi/vim、gedit、nano**

>vi/vim直接在命令行界面执行，显示简便，但是修改等操作不如后面两者简便
>>vi/vim异同点及操作：https://blog.csdn.net/qq_37896194/article/details/80369432   
>>vim操作 进入编辑后，按esc退出，然后输入     :wq    保存退出

>gedit可以打开类似txt的文本界面，方便修改，ubuntu可用。但是如果服务器虚拟机不支持图形显示，gedit则不可用
>>gedit的安装方法：sudo yum install gedit   
>>gedit为什么不能用？To use a gnome app you need a desktop. A putty session doesn't supply a desktop unless you install one on the system that you are running putty from. Use a text editor like vim or nano instead.不过没有关系，nano大法好！

>nano类似于在命令行界面显示文本界面，结合了vim和gedit，介于两者之间
>>nano操作教程：https://ipcmen.com/nano   
>>nano使用教程： ^x表示 ctrl+X

使得配置生效
```shell
. /etc/profile
```
查看变量
```
[root@localhost ~]#  echo $JAVA_HOME  
/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64

 [root@localhost ~]# echo $CLASSPATH
.:/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/lib/dt.jar:/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/lib/tools.jar
```
 javac 和java 命令都有输出设置提示就表示安装和环境配置成功了
 
 **具体案例见附录**
 
 

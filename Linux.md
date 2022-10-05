markdown [教程](https://blog.csdn.net/u014061630/article/details/81359144);
### Linux文件系统
在 Linux 操作系统中，所有被操作系统管理的资源，例如网络接口卡、磁盘驱动器、打印机、输入输出设备、普通文件或是目录都被看作是一个文件。 也就是说在 Linux 系统中有一个重要的概念：一切都是文件。
#### inode
* 存储文件的 元信息 metadata的区域 ：如某个文件被分成几块(4KB即8个扇区，每个扇区512字节)、每一块在的地址、文件拥有者，创建时间，权限，大小等。每个文件都有。
* 可以使用`stat`命令查看文件的inode信息，每个inode都有一个号码，系统用来区分不同的文件 
#### 文件类型
* 普通文件（-） ： 用于存储信息和数据， Linux 用户可以根据访问权限对普通文件进行查看、更改和删除。比如：图片、声音、PDF、text、视频、源代码等等。
* 目录文件（d，directory file） ：目录也是文件的一种，用于表示和管理系统中的文件，目录文件中包含一些文件名和子目录名。打开目录事实上就是打开目录文件。
* 符号链接文件（l，symbolic link） ：保留了指向文件的地址而不是文件本身。
* 字符设备（c，char） ：用来访问字符设备比如键盘。
* 设备文件（b，block） ： 用来访问块设备比如硬盘、软盘。
* 管道文件(p,pipe) : 一种特殊类型的文件，用于进程之间的通信。
* 套接字(s,socket) ：用于进程间的网络通信，也可以用于本机之间的非网络通信。
#### 常见目录说明
* /bin： 存放二进制可执行文件(ls、cat、mkdir 等)，常用命令一般都在这里；
* /etc： 存放系统管理和配置文件；
* /home： 存放所有用户文件的根目录，是用户主目录的基点，比如用户 user 的主目录就是/home/user，可以用~user 表示；
* /usr ： 用于存放系统应用程序；
* /opt： 额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把 tomcat 等都安装到这里；
* /proc： 虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息；
* /root： 超级用户（系统管理员）的主目录（特权阶级^o^）；
* /sbin: 存放二进制可执行文件，只有 root 才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如 ifconfig 等；
* /dev： 用于存放设备文件；
* /mnt： 系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统；
* /boot： 存放用于系统引导时使用的各种文件；
* /lib ： 存放着和系统运行相关的库文件 ；
* /tmp： 用于存放各种临时文件，是公用的临时文件存储点；
* /var： 用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件（系统启动日志等。）等；
* /lost+found： 这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows 下叫什么.chk）就在这里。
### Linux命令
[在线命令查找手册](https://www.w3xue.com/manual/linux/)
1. apt[^1] ***软件包管理***
   [^1]:分为`apt`,`apt-get`,`apt-cache`,其中`apt`集成了后两者的功能，并于16.04版本后被推荐使用。
    * 搜索安装包`sudo apt-cache search <name>`
    * 更新包索引，即更新包的数据库 `sudo apt-get update`
    * 升级软件包版本`sudo apt-get upgrade <package_name>`
    * 安装软件包` sudo apt-get install <package_name>`,不确定包名时，可使用Tab切换可能的包；
    * 删除软件包,但会保留配置文件`sudo apt-get remove <package_name>`
    * 删除软件包所有相关内容`sudo apt-get purge <package_name>`
    * 卸载所有自动安装且不再使用的软件包`sudo apt autoremove`
     <font size=1>\**详细内容参考`apt help`或`apt-get help`等*</font>
   
2. dpkg ***为‘Debian‘开发的软件包管理系统，需要手动解决依赖***
    * 手动安装(不包括下载)软件包`sudo dpkg -i <package_name>`，遇到依赖问题可使用`apt-get -f install`解决 
3. [wget](http://lnmp.ailinux.net/wget) ***从网络上进行下载***
    * 下载单个文件，并命名为最后一个‘/‘后的字符串`wget <url>`
    * 下载单个文件，并指定命名`wget -O <filename> <url>`
    * 使用后台下载，使用参数`-b`
4. [kill](http://t.zoukankan.com/zh-dream-p-12336812.html)[^2] ***删除执行中的程序或进程***
   [^2]:向操作系统内核发送一个信号（多是终止信号）和目标进程的 PID，然后系统内核根据收到的信号类型，对指定进程进行相应的操作。
    * 基本使用形式为`kill <signal> <pid>`
    * 挂起进程？（重启进程）`-1`
    * 强制结束进程`-9`
    * 正常结束进程`-15`
   <font size=1>\**详细内容参考`man kill`*</font>
5. killall ***根据名称'杀死'进程(<u>kill</u> processes by name)***
   * 杀死指定名字的所有进程`killall <name>`
   * 终止某个用户的进程`-u <username>`
   * 终止运行超过`-o <time>`或小于`-y <time>`某个时长的进程
  <font size=1>\**详细内容参考`man killall`*</font>
6. netstat ***监控TCP/IP网络的工具***
   * 显示进程名和pid`-p`
   * 只显示监听端口`-l`
   * 显示网络统计信息`-s`
   * 显示tcp`-t`或udp`-u`网络
   * 查找指定端口 `netstat -alnp | grep <端口>`
<font size=1>\**详细内容参考`man netstat`*</font>

#### 环境变量配制
Linux配制环境变量有多种方式，此处只介绍在profile中修改的两种方法，首先是直接在profile文件中修改，此处不做详细介绍`sudo vim /etc/profile`，其次是在/etc/profile.d目录下增加配制环境变量的shell文件，这两种方法均可做到所有用户有效。其具体流程如下：

1. 进入profile.d目录`cd /etc/profile.d`
2. 新建shell文件`sudo vim my_config.sh`,并写入环境变量配制，其中多个PATH用`:`分隔开
```shell
 export PATH=$PATH:/usr/local/ideaIU-2022.1.1/idea-IU-221.5591.52/bin：<PATH2>
```
3.  重新加载profile文件`source /etc/profile`,或重启计算机
该方法实现了配制的解耦合，可灵活更改系统配制，其可生效是由于profile文件中
```shell
if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi
```
对/etc/profile.d中的.sh文件进行遍历，实现环境变量的导入。

#### 配制ssh服务器
所需软件包openssh-server,通过`ps -e | grep ssh`查看服务器是否启动，可通过以下命令启动或重启服务器`/etc/init.d/ssh start`,`/etc/init.d/ssh restart`.
1. 查看主机ip地址`ifconfig`
2. 创建新用户`useradd <username> -ms /bin/bash`并修改密码`passwd <username>`
3. 连接ssh服务器`ssh <username>@<host_ip>`

#### 配制vpn
1. 没有L2TP协议,通过`sudo apt-get install network-manager-l2tp-gnome`下载，并重启后即可

#### 防火墙配制
1. 查看防火墙状态`sudo ufw status`,也可以看到开放的端口
2. 关闭防火墙`sudo ufw disable`，打开防火墙`sudo ufw enable`,
3. 防火墙开放端口`sudo ufw allow <socket>` ,关闭端口`sudo ufw deny <socket>`
4. 重启防火墙`sudo ufw reload`
#### 端口配制（iptables在最底层，一般都使用ufw）
1. 开启指定端口 `iptables -I INPUT -p tcp --dport 8080 -j ACCEPT`,但是服务器重启就会失效
2. 持续化规则，安装`iptables-persistent`,执行`netfilter-persistent save`，`netfilter-persistent reload`
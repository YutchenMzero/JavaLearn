markdown [教程](https://blog.csdn.net/u014061630/article/details/81359144);

###Linux命令

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

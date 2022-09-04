## 项目设计
### 需求分析
#### 需求获取
1. 学生用户应当可以注册并登录系统，
 


## 项目实现
### FTP服务器
#### 软件安装
1. 安装vsftp软件`sudo apt-get install vsftpd`
2. 设置软件为开机启动`sudo systemctl enable vsftpd`
3. 启动ftp服务`sudo systemctl start vsftpd `
4. 确认服务是否启动`sudo netstat -antup | grep ftp `
#### vsftp配置
1. 创建ftp用户，使用m参数，把ftp用户的home目录一起创建`sudo useradd -m ftpuser`,并设置密码`sudo passwd ftpuser`
2. 打开 vsftpd.conf 文件`sudo vim /etc/vsftpd.conf `,修改配置参数。常用配制如下
```sh
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES #是否限制用户离开限定的根目录
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list

listen=YES 
write_enable=YES
#listen_ipv6=YES
```
```sh
#开启被动模式
local_root= 本地用户登录后所在目录
allow_writeable_chroot=YES
pasv_enable=YES
pasv_address=xxx.xx.xxx.xx #请修改为您服务器公网 IP
#服务器建立数据传输可使用的端口范围值,此范围的端口和端口21需要开放
pasv_min_port=40000
pasv_max_port=45000 
```
3. 创建并编辑 chroot_list 文件,输入用户名，一个用户名占据一行,完成后重启ftp服务

 
#### 注意事项
1. vsftpd 默认开启匿名访问模式，无需通过用户名和密码即可登录 FTP 服务器。使用此方式登录 FTP 服务器的用户没有权修改或上传文件的权限
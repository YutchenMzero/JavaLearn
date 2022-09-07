## 项目设计
### 需求分析
#### 需求获取
1. 学生用户应当可以注册并登录系统，可以从系统上下载所需要的文件
2. 管理员账号应当可以管理系统中不同文件夹得访问权限，仅仅让拥有权限的人下载文件
3. 管理员应当可以审核用户的注册请求，仅能让内部人员访问该系统
4. 考虑到数据的特性，系统的传输应当能支持5g大小的文件传输
5. 对于文档、表格、图片等文件，并不是一定要下载下来，应当支持在线预览
6. 用户可以上传并管理自己上传的文件
7. 管理员可以查看存储端剩余存储空间，以及用户的敏感操作历史，主要是文件的上传与删除
8. 学生用户可以将自己上传的文件的从属权转让给管理员用户，便于统一管理
9. 用户可以选择自己上传的文件，其他用户包括管理员是否有操作权限（包括查看，删除，下载等）

#### 功能性需求
##### 用户基本功能
1. 注册
2. 登录
##### 文件操作
1. 文件上传（支持大文件）
2. 文件下载
3. 文件管理（移动，重命名，删除，更改权限）
##### 仓库管理
1. 查看剩余存储空间
##### 管理员（用户管理）功能
1. 设置用户存取权限
2. 查看操作日志？



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
2. 若ftp上传出现`FAIL MKDIR`错误，可通过以下流程解决：
   * 切换至ftp用户`su ftpuser`
   * 更改ftp目录权限`sudo chmod -R 777 ftp`


### MySQL
1. varchar(M) 在mysql5.0.3版本以下指存放M个字节的字符，5.0.3版本及以上表示存放M个字符。
2. varchar（100）和char（20）在存储“hello”时所用的空间一致，但前者在排序时所耗时间更长。

### Mybatis
#### 使用MyBatis Generator自动生成
#### 相关配制
1. pom.xml部分
```xml
<dependency>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-core</artifactId>
	<version>1.4.1</version>
</dependency>

<plugin>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-maven-plugin</artifactId>
	<version>1.4.1</version>
	<configuration>
	<!--配置文件的位置-->
	   <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
	   <verbose>true</verbose>
		<overwrite>true</overwrite>
   </configuration>
	<executions>
		<execution>
			<id>Generate MyBatis Artifacts</id>
			<goals>
				<goal>generate</goal>
			</goals>
	   </execution>
	</executions>
</plugin>

```

##### 注意事项
1. 自动生成 User 类的同时会生成 UserKey 及 UserWithBlob 类的问题：
   * 首先要在 <jdbcConnection> 中的 connectionURL 中指定数据库的实例名，
   * 然后在 <jdbcConnection> 中添加相关配置信息，即 <property name="nullCatalogMeansCurrent" value="true"/>，即可保证只生成自己需要的 User 类。

### 其他
#### MD5
1. MD5是一种摘要算法，加密后为128位，按16进制编码后，为32个字符。
2. 虽然md5本身是不可逆的,但是因为其唯一性,会遭到遍历破解,所以一般使用md5的时候还会进行加盐操作,即在md5运算结果上再加入自己的加密算法,例如这里为将结果都加上字符串yan,这样就不容易被破解了,加盐的加密算法也可以自己直接封装在工具类中
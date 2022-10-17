## Docker
### 基本概念
#### 镜像
类似与虚拟机镜像，可以将他理解为一个只读的模板，可以包含一个基本的操作系统，是创建Docker容器的基础。
#### 容器
类似于一个轻量级的沙箱，利用其来运行和隔离应用，它是从镜像创建的应用运行实例。且其互相时彼此相互隔离、互不可见的。其运行时会在镜像的最上层创建一个可见层。
#### 仓库
类似于代码仓库，是Docker集中存放镜像文件的场所。
### 使用镜像
#### 获取镜像
`docker [image] pull <name>[:TAG]`
1. 其中TAG常用来表示镜像的版本号，缺省时默认为`latest`
2. 若从非官方库下载，需要在仓库名前指定完整的仓库地址
3. 可以在启动配置中增加`--registry-mirror=proxy_URL`指定代理服务器地址
#### 查看镜像信息
1. `docker images`:列出镜像
2. `docker tag <image_name1> <image_name2>`: 添加镜像标签，实际上是相当于创建链接的作用
3. `docker [image] inspect <image_name>`查看详细信息，包括制作者、适应架构、各层的数字摘要等，返回为Json类型，使用`docker [image] inspect -f {{“.attribute”}} <image_name>`获取指定属性
4. `docker history <image_name>`查看镜像历史
#### 搜寻镜像
1. `docker search [option] <ketword>`搜索包含关键字的镜像
2. 关于`[option]`：
   * `-f` 过滤输出内容
   * `--no-trunc`不截断输出内容（默认限制为25）
#### 删除和清理
1. `docker rmi <image>`或`docker image rm <image>`其中`<image>`可以是标签或ID
   * 当指定的是标签时，只是会删除对应标签，除非此标签时最后一个
   * 当指定的是ID，会先尝试所有指向该镜像的标签，然后再删除镜像本身（除非其创建的容器存在）
   * `-f` 强制删除
   * `-no-prune` 不要清理未带标签的父镜像
2. `docker image prune [option]`清理临时及未使用镜像
   * `-a` 清除所有
   * `-filter <filter>` 指定过滤器
   * `-f` 强制删除，不进行确认
#### 创建
1. `docker [container] commit [option] <container_name> [<repository:[tag]>]` 基于已有镜像的容器创建
   * 可用ID或者名字指定容器
   * `-a <" ">`作者信息
   * `-c`提交时执行Dockerfile指令
   * `-m`提交信息
   * `-p`提交时暂停容器运行
2. `docker [container] import [option] <file | url> [<repository:[tag]>]`基于本地模板导入
3. Dockerfile形式
#### 导出与载入
1. `docker [image] save [-o <output_name>] <image_name>`导出镜像（到指定文件）
2. `docker [image] load [-i <input_name>]`或`docker [image] load [< <input_name>]`从指定文件中读入镜像内容
#### 上传
1. `docker push <image_name[:tag]>`默认上传到官方仓库，需要登录

### 操作容器
#### 创建容器
1. `docker [container] create [option] <image_name>`根据镜像创建实例。常用配制如下
   * `-i` 保持标准输入打开
   * `-t` 分配伪终端
   * `--name` 指定容器别名
   * `-d` 在后台运行
2. `docker [container] start <container_name>`启动容器
3. `docker [container] run [option] <image_name>`创建并启动容器某些时候，执行此条时因为命令无法正常执行会出错直接退出，常见的错误代码为：
   * `125` 指定了不支持的docker命令参数
   * `126` 指定命令无法执行，如权限不足
   * `127` 容器内命令无法找到
#### 停止容器
1. `docker [container] pause <container_name>`暂停容器的运行，可使用`unpause`指令恢复
2. `docker [container] stop [-t = 10] <container_name>`终止容器，先尝试用SIGTERM信号，等待一定时间后使用SIGKILL信号，使用`kill`直接使用SIGKILL信号。可使用`start`来重新启动
#### 进入容器
当使用`-d`参数时，容器进入后台运行，用户无法看到容器中的信息也无法进行操作，需要进入容器
1. `docker [container] attach [option] <container_name>`
   * `–detach-keys`	指定退出attach模式的快捷键序列，默认是 CTRL-p
   * `–no-stdin`	是否关闭标准输入，默认是打开
   * `–sig-proxy` 是否代理收到的系统信号给应用进程，默认为true
   * 多个窗口同时attach到同一个容器时，所有窗口都会同步显示，某个窗口阻塞则其他窗口也无法操作
2. `docker [container] exec [option] <container_name> [<command>]`
   * `-d` :分离模式,在后台运行
   * `-i` :即使没有附加也保持STDIN 打开
   * `-t` :分配一个伪终端
   * `--privilegd true | false` ：是否给执行命令以高权限，默认为false
   * `--detach-keys ""` :指定将容器切回后台的按键
   * `-e []` :指定环境变量列表
   * `-u ""` :执行命令的用户名或ID
#### 删除容器
1. `docker [container] rm [option] <container_name>`删除处于终止或退出状态的容器
   * `-f`强行删除运行时容器：先发送SIGKILL信号，再删除。
   * `-l`删除容器的连接但保留容器
   * `-v`删除容器挂载的数据卷
#### 导入和导出
1. `docker [container] export [-o ""] <container_name>`将一个任意状态的已创建容器导出到一个文件，也可使用重定向指定导出的tar文件
2. `docker [container] import <file|url> <repository>`将容器导入到本地镜像库（会舍弃历史信息和元数据）
#### 查看容器
1. `docker [container] inspect [option] <container_name>`查看容器详情，以json形式返回
2. `docker [container] top [option] <container_name>`类似于top命令，会打印处容器内的进程信息
3. `docker [container] stats [option] <container_name>`显示CPU、内存、存储等统计信息
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
3. `docker [container] run [option] <image_name>`创建并启动容器
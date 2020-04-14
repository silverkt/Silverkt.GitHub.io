## 概念
#### Host（Docker 宿主机）
```
安装了Docker程序，并运行了Docker daemon的主机。
```

#### Docker daemon(守护进程)：
```
运行在宿主机上，Docker守护进程，用户通过Docker client(Docker命令)与Docker daemon交互。
```

#### Images（镜像）：
```
将软件环境打包好的模板，用来创建容器的，一个镜像可以创建多个容器。
```

#### Containers（容器）：
```
Docker的运行组件，启动一个镜像就是一个容器，容器与容器之间相互隔离，并且互不影响。
```

#### Docker Client(客户端)
```
Docker命令行工具，用户是用Docker Clients与Docker daemon进行通信并返回结果给用户。也可以使用其他工具通过Docker Api与Docker daemon通信。
```

#### Registry(仓库服务注册器)
```
经常会和仓库(Repository)混为一谈，实际上Registry上可以有多个仓库，每个仓库可以看成是一个用户， 一个用户的仓库放了多个镜像。仓库分为了公开仓库(Public Repository)和私有仓库(Private Repository)，最大的公开仓库是官方的Docker Hub，国内也有如阿里云、时速云等，可以给国内用户提供稳定快速的服务。用户也可以在本地网络内创建一个私有仓库。当用户创建了自己的镜像之后就可以使用push命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上pull下来就可以了。
```


##  基础应用

#### docker 查看本机容器 

```bash
docker ps -a
```

#### docker 查看本机镜像 

```bash
docker images
```
#### docker save : 将指定镜像保存成 tar 归档文件。
```bash
docker save -o 目标文件名.tar 镜像名或者id
```

#### docker  导入已保存压缩tar镜像

```bash
docker load -i imagesample.tar
```

#### docker 创建容器但不启动它

```bash
docker create --name myimgname nginx:latest
```

#### docker tag 修改

```bash
可以使用命令: docker tag [image id] [name]:[版本]

例如: docker tag b03b74b01d97 docker-redis:0.0.1
```

#### docker 使多端口开放

```bash
docker run -it -p20180:80   -p20181:8080  -p20182:8976 --name containerName image:tag

附提交

docker commit -a ‘songlk’ -m ‘commitdesc’ container_nginx centos:7.2.1511 
```

## 常用操作

输入`docker`可以查看Docker的命令用法，输入`docker COMMAND --help`查看指定命令详细用法。

#### 镜像操作

查找镜像：

```bash
# 搜索docker hub网站镜像的详细信息
docker search 关键词
```

下载镜像：

```bash
# Tag表示版本，有些镜像的版本显示latest，为最新版本
docker pull 镜像名:TAG
```

查看镜像：

```bash
# 查看本地所有镜像
docker images
```

删除镜像：

```bash
# 删除指定本地镜像
docker rmi -f 镜像ID或者镜像名:TAG

# -f 表示强制删除
```

获取元信息：

```bash
# 获取镜像的元信息，详细信息
docker inspect 镜像ID或者镜像名:TAG
```

#### 容器操作

运行：

```bash
docker run --name 容器名 -i -t -p 主机端口:容器端口 -d -v 主机目录:容器目录:ro 镜像TD或镜像名:TAG

# --name 指定容器名，可自定义，不指定自动命名
# -i 以交互模式运行容器
# -t 分配一个伪终端，即命令行，通常组合来使用
# -p 指定映射端口，将主机端口映射到容器内的端口
# -d 后台运行容器
# -v 指定挂载主机目录到容器目录，默认为rw读写模式，ro表示只读

windows 示例：
docker run --name 起一个容器名 -p 80:80 -p 443:443 -d -v D:\dkfolder:/usr/share/nginx 容器id
```

容器列表：

```bash
# docker ps 查看正在运行的容器
docker ps -a -q

# -a 查看所有容器(运行中、未运行)
# -q 只查看容器的ID
```

启动容器：

```bash
docker start 容器ID或容器名
```

停止容器：

```bash
docker stop 容器ID或容器名
```

删除容器：

```bash
docker rm -f 容器ID或容器名

# -f 表示强制删除
```

查看日志：

```bash
docker logs 容器ID或容器名
```

进入正在运行容器：

```bash
docker exec -it 容器ID或者容器名 /bin/bash

# 进入正在运行的容器并且开启交互模式终端

# /bin/bash是固有写法，作用是因为docker后台必须运行一个进程，否则容器就会退出，在这里表示启动容器后启动bash。

# 也可以用docker exec在运行中的容器执行命令
```

拷贝文件：

```bash
docker cp 主机文件路径 容器ID或容器名:容器路径 # 主机中文件拷贝到容器中

docker cp 容器ID或容器名:容器路径 主机文件路径 # 容器中文件拷贝到主机中
```

获取容器元信息：

```bash
docker inspect 容器ID或容器名
```

#### 创建镜像

有时候从Docker镜像仓库中下载的镜像不能满足要求，我们可以基于一个基础镜像构建一个自己的镜像。

两种方式：

- 更新镜像：使用`docker commit`命令
- 构建镜像：使用`docker build`命令，需要创建Dockerfile文件

#### 更新镜像

先使用基础镜像创建一个容器，然后对容器内容进行更改，然后使用`docker commit`命令提交为一个新的镜像（tomcat为例）。

1. 根据基础镜像，创建容器

```css
docker run --name mytomcat -p 80:8080 -d tomcat
```

1. 修改容器内容

```bash
docker exec -it mytomcat /bin/bash

cd webapps/ROOT

rm -f index.jsp

echo hello world > index. html

exit
```

1. 提交为新镜像

```bash
docker commit -m="描述消息" -a="作者" 容器ID或容器名 镜像:TAG

# 例：
# docker commit -m="修改了首页" -a="测试" mytomcat zong/tomcat:v1.0
```

1. 使用新镜像运行容器

```bash
docker run --name tom -p 8080:8080 -d zong/tomcat:v1.0
```

#### 使用Dockerfile构建镜像

Dockerfile是一个包含创建镜像所有命令的文件，使用`docker build`命令可以根据Dockerfile的内容创建镜像(以tomcat为例)。

1. 创建一个Dockerfile文件 `vi Dockerfile`

```bash
#注意dockerfile指令须大写

FROM tomcat

MAINTAINER zong

RUN rm -f /usr/local/tomcat/webapps/ROOT/index.jsp

RUN echo "<h1>hello world2<h1>" > /usr/local/tomcat/webapps/ROOT/index.html
```

1. 构建新镜像

```bash
docker build -f Dockerfile -t zong/tomcat:v2.0 .

# -f Dockerfile路径，默认是当前目录
# -t 指定新镜像的名字以及TAG
```

#### 使用Dockerfile构建Spring Boot应用镜像

 —、准备

1. 把你的spring boot项目打包成可执行jar包
2. 把jar包上传到Linux服务器

二、构建

1. 在jar包路径下创建Dockerfile文件`vi Dockerfile`

```bash
# 指定基础镜像，本地没有会从dockerHub pull下来
FROM java:8

# 可执行jar何复制到基础镜像的根目录下
ADD test.jar /test.jar

# 镜像要暴露旳端口，如要使用端口，在执行docker run命令时使用-p生效
EXPOSE 8080

# 在镜像运行为容器后执行旳命令
ENTRYPOINT ["java", "-jar", "/test.jar"]
```

1. 使用`docker build`命令构建镜像，基本语法

```bash
docker build -f Dockerfile -t zong/mypro:v1 .

# -f 指定Dockerfile文件的路径
# -t 指定镜像名字和TAG
# . 指当前目录，这里实际上需要一个上下文路径
```

三、运行

运行自己的Spring Boot镜像

```bash
docker run --name pro -p 80:80 -d 镜像名:TAG
```



## 扩展阅读

#### Docker启动就退出的解决方案

**现象**

启动docker容器

```
docker run –name [CONTAINER_NAME] [CONTAINER_ID]
```

查看容器运行状态

```
docker ps -a
```

发现刚刚启动的mydocker容器已经退出

**原因**

很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程.

容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的

docker容器的主线程（dockfile中CMD执行的命令）结束，容器会退出

**解决方法**

可以使用交互式启动

```
docker run -i [CONTAINER_NAME or CONTAINER_ID]
```

上面的不太友好，建议使用后台模式和tty选项

```
docker run -dit [CONTAINER_NAME or CONTAINER_ID]
```

docker调出后台容器

```
docker attach [CONTAINER_NAME or CONTAINER_ID]
```

TIPs:退出时，使用[ctrl + D]，这样会结束docker当前线程，容器结束，可以使用[ctrl + P][ctrl + Q]退出而不终止容器运行

如下命令，会在指定容器中执行指定命令，[ctrl+D]退出后不会终止容器运行

```
docker  exec -it [CONTAINER_NAME or CONTAINER_ID]  /bin/bash
```
**其他方式**
将要运行的程序以前台进程的形式运行，如果容器需要同时启动多个进程，那么也只需要将其中一个挂起到前台即可。
比如上面所说的 web 容器，只需要将启动指令修改为:

```
service php5-fpm start && nginx -g "daemon off;"
```
投机方案
对于可能不知道怎么前台运行的程序，提供一个投机方案，只需要在启动的命令之后，添加类似于 tail top 这种可以前台运行的程序，这里推荐tail ，然后持续输出log文件即可。
```
service nginx start && service php5-fpm start && tail -f /var/log/nginx/error.log
```
再以上面所说的 web 容器为例，可以写成：
```
service nginx start && service php5-fpm start && tail -f /var/log/nginx/error.log
```
以centos/ubuntu为例
同上，在启动centos/ubuntu容器时，可以做一个手脚：做一个死循环，持续输出任意，这样容器不会认为没事可做而自杀了。
```
docker run -d centos /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

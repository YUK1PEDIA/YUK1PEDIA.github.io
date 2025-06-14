---
layout: post
title: Docker
description: 记录
tag: 笔记
---

## 1.镜像和容器

利用 Docker 安装应用时，Docker 会自动搜索并下载应用**镜像（image）**。镜像不仅包含应用本身，还包含应用所需要的环境、配置、系统函数库（此处的环境是**最轻量的**，只包含容器内运行需要的环境，其他编辑操作可能包含）。Docker 会在运行镜像时创建一个隔离环境，称为**容器（container）**。

**镜像仓库**：存储和管理镜像的平台，Docker 官方维护了一个公共仓库：https://hub.docker.com/



## 2.命令解读

```dockerfile
docker run -d \
	--name mysql \
	-p 3306:3306 \
	-e TZ=Asia/Shanghai \
	-e MYSQL_ROOT_PASSWORD=123 \
	mysql
```

- `docker run` ：创建并运行一个容器，**-d** 是让容器在后台运行。
- `--name mysql` ：给容器起个名字，**必须唯一**。
- `-p 3306:3306` ：设置端口映射，docker 创建的容器拥有独立的文件系统、内存、网络空间等，从外部直接对 docker 容器进行 ping 操作是 ping 不通的，需要设置服务器或虚拟机的端口映射，才能通过服务器或虚拟机访问到上面的 docker 容器。**冒号前面的端口是宿主机端口，冒号后面是容器内端口**。
- `-e KEY=VALUE` ：设置环境变量。上面的命令中，`TZ=Asia/Shanghai` 是配置时区
- `mysql` ：指定运行的镜像的名字



**补充：镜像命名规范**

- 镜像名称一般分两部分组成：`[repository]:[tag]` 
  - 其中 *repository* 就是镜像名
  - *tag* 是镜像的版本
- 在没有指定 *tag* 时，默认是 *latest* ，代表最新版本的镜像





## 3.常见命令

![1.jpg](https://s2.loli.net/2024/10/08/SzMIaOdynwvUPQh.jpg)

- `docker rmi` 是删除镜像，`docker rm` 是删除容器
- `docker run` 是**创建并运行容器**，这条命令会创建一个新的容器，如果要启动已经有的容器，需要执行 `docker start` 命令
- `docker exec` 是进入到容器内部





## 4.数据卷挂载

### 数据卷挂载

**数据卷（volume）**是一个虚拟目录，是**容器内目录**与**宿主机目录**之间映射的桥梁。

![1.jpg](https://s2.loli.net/2024/10/08/sG4LVg3XIKkTiBF.jpg)

- 在执行 `docker run` 命令时，使用 `-v 数据卷:容器内目录` 可以完成数据卷挂载

  例：`docker run -d --name nginx -p 80:80 -v html:/user/share/nginx/html nginx`

- 当创建容器时，如果挂载了数据卷且数据卷不存在，会自动创建数据卷

数据卷挂载后，我们**可以直接在宿主机进行文件的编辑操作**，不用进入到容器中下载需要的编辑工具再对文件进行编辑。



数据卷的常见命令如下：

- `docker volume ls` ：查看数据卷
- `docker volume rm` ：删除数据卷
- `docker volume inspect` ：查看数据卷详情
- `docker volume prune` ：删除未使用的数据卷





### 本地挂载

我们进行数据卷挂载时，可以不通过数据卷名称进行挂载。

只用将之前的 `docker run` 命令里的 `-v 数据卷:容器内目录` 换成 `-v 本地目录:容器内目录` 即可完成本地目录挂载

- 本地目录必须以 "/" 或 "./" 开头，**如果直接以名称开头，会被识别为数据卷而非本地目录**
  - `-v mysql:/var/lib/mysql` 会被识别为一个数据卷叫 *mysql*
  - `-v ./mysql:/var/lib/mysql` 会被识别为当前目录下的 `mysql` 目录





## 5.自定义镜像

镜像就是包含了应用程序、程序运行的系统函数库、运行配置等文件的文件包。构建镜像的过程其实就是把上述文件打包的过程。

### 

### Dockerfile

Dockerfile是一个文本文件，其中包含一个个的指令，用指令来说明要执行什么操作来构建镜像。将来 Docker 可以根据 Dockerfile 帮我们构建镜像。常见指令如下：

| 指令       | 说明                                           | 示例                                                         |
| ---------- | ---------------------------------------------- | ------------------------------------------------------------ |
| FROM       | 指定基础镜像                                   | FROM centos:6                                                |
| ENV        | 设置环境变量，可在后面指令使用                 | ENV key value                                                |
| COPY       | 拷贝本地文件到镜像的指定目录                   | COPY ./jrell.tar.gz /tmp                                     |
| RUN        | 执行 Linux 的 shell 命令，一般是安装过程的命令 | RUN tar -zxvf /tmp/jrell.tar.gz && EXPORTS path=/tmp/jrell:$path |
| EXPOSE     | 指定容器运行时监听的端口，是给镜像使用者看的   | EXPOSE 8080                                                  |
| ENTRYPOINT | 镜像中应用的启动命令，启动运行时调用           | ENTRYPOINT java -jar xx.jar                                  |

![1.jpg](https://s2.loli.net/2024/10/08/MRIDxjaAP7ztBQK.jpg)



编写好了 Dockerfile ，可以利用下面命令来构建镜像：

`docker build -t myImage:1.0 .`

- `-t` ：是给镜像起名，格式依然是 `repository:tag` 的格式，不指定 *tag* 时，默认为 *latest*
- `.` ：是指定 Dockerfile 所在目录，如果就在当前目录，则指定为 "."





## 6.容器网络互连

![1.jpg](https://s2.loli.net/2024/10/08/n1MKjAk87w6LHEa.jpg)

![1.jpg](https://s2.loli.net/2024/10/08/J5kC8m6wjELIdor.jpg)




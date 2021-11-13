# Docker的学习笔记

## Docker的介绍

### 1、抛出问题

```bat
问题1:
	某IT部门要上线一个项目。常规操作，直接去线上服务器，拷贝一个tomcat，然后改端口号，然后部署应用到webapps文件夹下，重启就好。
	一个服务器上可能会部署多个应用服务。如果某个应用出现问题，CPU100%，可能这个服务器上的其他应用也会出现问题。
	对于一个大型应用拆分为几十个微服务，分别交由不同的团队开发，不同团队之间水平参差不齐。如果还采用这种部署方式，你的应用可能会因为另一个团队的应用发生意外。因部署在了同一台服务器上，导致全部出现问题。  服务器内的微服务耦合性会很高
```


```bat
问题2:
	开发和线上代码（同一套代码）问题。开发阶段部署一套软件环境，测试人员在开发中测试没有问题，运维进行部署。但是正式部署到服务器时，发生了问题(启动参数、环境问题、漏配了参数)等意外。    可以统一定制一个镜像来解决
```


```bat
问题3:
	随着微服务技术的兴起，一个大的应用需要拆分成多个微服务。多个微服务的生成，就会面临庞大系统的部署效率，开发协同效率问题。然后通过服务的拆分，数据的读写分离、分库分表等方式重新架构，而且这种方式如果要做的彻底，需要花费大量人力物力。可能需要部署很多个服务器。
```
```bat
问题4:
	持续的软件版本发布/测试项目。到线上环境的集成
```

### 2、什么是docker

Docker是一个开源的应用容器引擎，基于==Go语言==并遵从Apache2.0协议开源。

Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似iPhone的app）,更重要的是容器性能开销极低。  ==容器之间耦合性低==

### 3、为什么用docker

#### 1、简化程序：

​		Docker让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是Docker的最大优势，过去需要用数天乃至数周的任务，在Docker容器的处理下，只需要数秒就能完成。

#### 2、避免选择恐惧症：

​		如果你有选择恐惧症，还是资深患者。Docker帮你打包你的纠结！比如Docker镜像；Docker镜像中包含了运行环境和配置，所以Docker可以简化部署多种应用实例工作。比如Web应用、后台应用、数据库应用、大数据应用比如Hadoop集群、消息队列等等都可以打包成==一个镜像(类)==部署。

#### 3、节省开支

​		一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker改变了高性能必然高价格的思维定势。Docker与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

#### 4、持续交付和部署

​	   对开发和运维（DevOps）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。使用Docker可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过Dockerfile来进行镜像构建，并结合持续集成(ContinuousIntegration)系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合持续部署(ContinuousDelivery/Deployment)系统进行自动部署。而且使用Dockerfile使镜像构建透明化，不仅仅开发团队可以理解应用运行环境，也方便运维团队理解应用运行所需条件，帮助更好的生产环境中部署该镜像。

#### 5、更轻松的迁移

​       由于Docker确保了执行环境的一致性，使得应用的迁移更加容易。Docker可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

### 4、Docker的应用场景

Web应用的自动化打包和发布。

自动化测试和持续集成、发布。

在服务型环境中部署和调整数据库或其他的后台应用。

从头编译或者扩展现有的OpenShi或CloudFoundry平台来搭建自己的PaaS环境。

```bat
IaaS:（Infrastructure-as-a-Service）(基础设施即服务)
PaaS:（PlatformasaService）(平台即服务)
SaaS:（Software-as-a-Service）(软件即服务)
```



### 5、Docker和虚拟机总结

![image-20211103173938606](http://cdn.nefu-yzk.top/img/image-20211103173938606.png)

```bat
名词解释：
infrastructure （基础服务）硬件HostOS主机操作系统
VM虚拟机  Hypervisor 虚拟层程序
```



1.**实现原理技术不同**   虚拟机是用来进行硬件资源划分的完美解决方案，利用的是==硬件虚拟化技术==，如此VT-x、AMD-V会通过一个hypervisor层来实现对资源的彻底隔离。而容器则是操作系统级别的虚拟化，利用的是内核的Cgroup和Namespace特性，此功能通过软件来实现，仅仅是进程本身就可以实现互相隔离，不需要任何辅助。
2.**使用资源方面不同**   Docker容器与主机共享操作系统内核，不同的容器之间可以共享部分系统资源，因此更加轻量级，消耗的资源更少。虚拟机会独占分配给自己的资源，不存在资源共享，各个虚拟机之间近乎完全隔离，更加重量级，也会消耗更多的资源。
3.**应用场景不同**   若需要资源的完全隔离并且不考虑资源的消耗，可以使用虚拟机。若是想隔离进程并且需要运行大量进程实例，应该选择Docker容器。

虚拟机资源完全隔离，Docker存在部分共享资源(数据卷的形式)

![image-20211103174222418](http://cdn.nefu-yzk.top/img/image-20211103174222418.png)

### 6、Docker总结

- Docker是世界领先的软件容器平台。

- Docker使用Google公司推出的Go语言进行开发实现，基于Linux内核的cgroup，namespace，以及AUFS类的UnionFS等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。Docke最初实现是基于LXC。

- Docker能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在真正重要的事情上：构建杰出的软件。

- 用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

- Docker的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性，从而不会再出现“这段代码在我机器上没问题啊”这类问题；——一致的运行环境

- 可以做到秒级、甚至毫秒级的启动时间。大大的节约了开发、测试、部署的时间。——更快速的启动时间

- 避免公用的服务器，资源会容易受到其他用户的影响。——隔离性

- 善于处理集中爆发的服务器使用压力；——弹性伸缩，快速扩展

- 可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。——迁移方便

- 使用Docker可以通过定制应用镜像来实现持续集成、持续交付、部署。——持续交付和部署

## Docker的安装

### 1、简介

Docker使用客户端-服务器(C/S)架构模式，使用远程API来管理和创建Docker容器。

Docker容器通过Docker镜像来创建。  容器之间互不干扰

容器与镜像的关系类似于面向对象编程中的对象与类。

对象->容器      镜像->类

通过镜像来创建容器

### 2、Docker基本概念

Docker包括三个基本概念

```bat
镜像（Image）：相当于类
容器（Container）: 相当于实例
仓库（Repository）：Docker集中存放镜像文件的地方
```
理解了这三个概念，就理解了Docker的整个生命周期。
| Docker | 面向对象 |
| :----: | :------: |
|  容器  |   对象   |
|  镜像  |    类    |

### 3、Docker引擎

![image-20211103174525426](http://cdn.nefu-yzk.top/img/image-20211103174525426.png)

Docker使用客户端-服务器(C/S)架构模式，使用远程API来管理和创建Docker容器。

Docker容器通过Docker镜像来创建。

### 4、安装Docker

#### 1、移除旧的版本

```bat
sudo yum remove docker \ 
                docker-client \
				docker-client-latest \
				docker-common \
				docker-latest \
				docker-latest-logrotate \
				docker-logrotate \
				docker-selinux \
                docker-engine-selinux \
                docker-engine
                
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
# 如果 yum 报告未安装这些软件包，则可以。
```

![image-20211103191032781](http://cdn.nefu-yzk.top/img/image-20211103191032781.png)

#### 2、安装一些必要的系统工具

> 安装所需的软件包。 ==yum-utils== 提供了 ==yum-config-manager== 应用，并 ==device-mapper-persistent-data== 和 ==lvm2== 由需要 ==devicemapper== 存储驱动程序。

```bat
# 执行一下命令即可
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

![image-20211103191111923](http://cdn.nefu-yzk.top/img/image-20211103191111923.png)

#### 3、添加软件源信息

```bat
# 添加软件源可以加快下载速度，因为默认的镜像下载是国外的链接。
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#  sudo的单词为super user do  表示以超级管理员的权限执行相关语句
#  su的单词为switch user   表示切换用户
```

![image-20211103191133486](http://cdn.nefu-yzk.top/img/image-20211103191133486.png)

#### 4、更新 yum 缓存：

```bat
sudo yum makecache fast
```

![image-20211103191153224](http://cdn.nefu-yzk.top/img/image-20211103191153224.png)

#### 5、安装 Docker-CE

```sh
sudo yum -y install docker-ce
```

![image-20211103191328136](http://cdn.nefu-yzk.top/img/image-20211103191328136.png)

#### 6、启动 Docker 后台服务

```bat
sudo systemctl start docker
```

![image-20211103191421768](http://cdn.nefu-yzk.top/img/image-20211103191421768.png)

#### 7、重启Docker服务

```sh
sudo systemctl restart docker 
```

#### 8、查看docker版本号

```sh
docker version
```

未启动dokcer前查看version

![image-20211103191356013](http://cdn.nefu-yzk.top/img/image-20211103191356013.png)

启动docker后查看version

![image-20211103191440246](http://cdn.nefu-yzk.top/img/image-20211103191440246.png)

#### 9、卸载

```sh
# 执行以下的命令来卸载Docker CE
sudo yum remove docker-ce
sudo rm -rf /var/lib/docker
```

#### 10、配置加速器

> 鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决

通过命令查看：在 ==/etc/docker/daemon.json== 中写入如下内容（如果文件不存在请新建该文件）

```json
{
    # 这个是docker的官网加速器，一般不怎么快
	"registry-mirrors":["https://registry.docker-cn.com"]
}

#可以使用下面这个，比较快一些
{
	"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/","https://hub-mirror.c.163.com","https://registry.docker-cn.com"],
	"insecure-registries": ["10.0.0.12:5000"]
}

```

> 注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动

配置完成后需要重启Docker服务

```sh
sudo systemctl daemon-reload
# daemon-reload: 重新加载某个服务的配置文件，使修改的配置文件生效
sudo systemctl restart docker

docker info
# 查看镜像地址与所配置的镜像地址是否匹配，如过匹配，则说明配置成功
```

![image-20211103192505617](http://cdn.nefu-yzk.top/img/image-20211103192505617.png)



## Docker常用命令

```bat
docker pull nginx  # 表示从docker库中下载镜像
docker images   #查看docker下载的镜像内容 
# docker pull tomcat:版本号 //不写版本号 代表latest版本
```

![image-20211103193916250](http://cdn.nefu-yzk.top/img/image-20211103193916250.png)

```bat
# 运行tomcat及相关命令详解
docker pull tomcat:8  # 下载tomcat 8的镜像
docker run --rm -d --name tomcat-8081 -p 8081:8080 tomcat:8
# -d 表示后台运行
# --name 给即将运行的容器命名 不指定则随机命名
# tomcat:8  前面的tomcat表示镜像名称。后面的8表示版本号 
# 未指定版本号时，则默认使用latest的
# -p 指定端口号  宿主机端口号:服务器内部对应的端口号
# 端口映射，前面为宿主机的端口，后面为容器服务进程端口，访问宿主机的8080,最终会转发给容器的8080端口
# --rm 表示退出容器后，容器会被删除，常用于测试
```

![image-20211103184113060](http://cdn.nefu-yzk.top/img/image-20211103184113060.png)

> 运行tomcat后出现访问时出现上面的页面则证明tomcat启动成功，因为docker为了节省资源，所以并没有ROOT/index.html文件
>
> 其中的192.168.59.12 为我自己的linux虚拟机的ip地址

```bat
# 查看容器的两个命令
docker ps  # 表示只查看运行的容器
docker ps -a  # 表示查看所有的容器(包括运行和退出的)
docker container ls
docker container ls -a
```

![image-20211103185225892](http://cdn.nefu-yzk.top/img/image-20211103185225892.png)

```bat
docker images # 查看所有的镜像
docker image ls # 与docker images是同样的效果
```

![image-20211103185350762](http://cdn.nefu-yzk.top/img/image-20211103185350762.png)

> 列表包含了仓库名、标签、镜像 ID、创建时间 以及 所占用的空间。 标签即是表示版本号，镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个标签。因此，如果拥有相同的 ID，因为它们对应的是同一个镜像。



```bat
镜像体积 
    如果仔细观察，会注意到，这里标识的所占用空间和在 Docker Hub 上看到的镜像大小不同。这是因为Docker Hub中显示的体积是压缩后的体积。在镜像下载和上传过程中镜像是保持着压缩状态的，因此Docker Hub所显示的大小是网络传输中更关心的流量大小。而 docker image ls 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。另外一个需要注意的问题是，docker image ls 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于Docker镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。  一般而言镜像的大小要比实际所展示的小很多，因为有可能有共用的层
```



```bat
虚悬镜像
    镜像列表中，还可以看到一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为none，这个镜像原本是有镜像名和标签的，原来为 tomcat:8.0，随着官方镜像维护，发布了新版本后，重新 docker pull tomcat:8.0 时，tomcat:8.0 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了none。除了 docker pull可能导致这种情况，docker build 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为none的镜像。这类无标签镜像也被称为虚悬镜像(danglingimage) 。一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除：
  docker image prune   # 删除虚悬镜像(没有name和tag)
```



```bat
docker images -q # 查看本地的镜像ID
# -q 表示查看id(容器的或者是镜像的)
docker history image_name # 查看image_name(想要查看的镜像名称)的制作过程 
docker image rmi 镜像ID 
# 要删除镜像必须确认此镜像⽬前没有被任何容器使⽤
docker ps -a  # 查看有哪些容器(运行与退出)
docker ps  # 查看正在运行的容器
docker container ls -a # 同理
docker stop container_id   # 通过docker ps找到运行着的container_id,然后stop停止
docker container stop container_id/container_name #通过id或者name进行停止
docker start container_id/container_name # 启动已经停止的container
docker restart container_id/container_name # 重启已终止的容器
docker rm container_id # 关闭和删除容器

# 使用-d启动容器后，此时容器在后台启动，可以通过相关命令查看日志
docker logs tomcat-8081

docker logs -f -t --since="2018-12-1" --tail=10 tomcat-8081
# --since指定了输出日志的开始日期，即只会输出在该日期之后的日志
# -f 表示查看实时日志
# -t 表示查看日志产生的日期
# --tail=10 表示查看最后的十条，数字可以自己指定
```

### 1、Docker进入容器

某些时候需要进入容器进行操作，使用 ==docker exec== 命令

> -i -t 参数 docker exec 后边可以跟多个参数，这里主要说明 -i -t 参数。 只用 -i 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。 当 -i -t 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bat
docker exec -it container_id/name bash
# 上面表示进入容器  -it的作用主要是可以看到linux的提示符
# 一定要注意，为了节省内存，有些Linux命令无法在容器内使用，如vim，ll等
```

![image-20211104104840057](http://cdn.nefu-yzk.top/img/image-20211104104840057.png)

```bat
# 进入容器，在webapps中创建ROOT目录，然后进入ROOT目录，创建一个index页面
echo 'nefu-yzk.top'>>index.jsp
# 此时即可正常的进行访问了
# 使用exit命令退出容器内部
```

![image-20211104105224765](http://cdn.nefu-yzk.top/img/image-20211104105224765.png)



###  2、在宿主机和容器之间交换⽂件

在宿主机和容器之间相互COPY⽂件 cp的⽤法如下

```bat
# 容器中 复制到 宿主机
docker cp [OPTIONS] CONTAINER:PATH LOCALPATH 
eg：docker cp  tomcat-8081:/usr/local/tomcat/webapps/ROOT/index.jsp /root/

# 宿主机 复制到 容器中
docker cp [OPTIONS] LOCALPATH|- CONTAINER:PATH
eg：docker cp /root/index.html tomcat-8081:/usr/local/tomcat/webapps/ROOT/
```



如图所示，可以同时进行部署多个项目，相比之下更为简单

![image-20211104111034971](http://cdn.nefu-yzk.top/img/image-20211104111034971.png)

部署的结果

![image-20211104111130784](http://cdn.nefu-yzk.top/img/image-20211104111130784.png)

### 3、Docker数据卷

> ​		通过镜像创建一个容器。容器一旦被销毁，则容器内的数据将一并被删除。但有些情况下，通过服务器上传的图片出会丢失。容器中的数据不是持久化状态的。这个时候可以通过数据卷来解决这个问题。



#### 3.1什么是数据卷

==数据卷是一个可供一个或多个容器使用的特殊目录==

> 数据卷的特性：
>
> ​       数据卷可以在容器之间共享和重用
>
> ​       对数据卷的修改会立马生效
>
> ​       对数据卷的更新，不会影响镜像
>
> ​		数据卷默认会一直存在，即使容器被删除

![image-20211104111534716](http://cdn.nefu-yzk.top/img/image-20211104111534716.png)

通俗一点而言，数据卷相当于硬盘，可以保存服务器的数据，即是服务器关闭了，数据依然不会丢失。  数据卷是一种映射关系，即容器内部的文件夹与宿主机中的文件夹相关映射(关联)

#### 3.2 为什么需要数据卷

> 这得从 docker 容器的文件系统说起。出于效率等一系列原因，docker 容器的文件系统在宿主机上存在的方式很复杂，这会带来下面几个问题： 
>
> -- 不能在宿主机上很方便地访问容器中的文件。
>
> -- 无法在多个容器之间共享数据。
>
> -- 当容器删除时，容器中产生的数据将丢失。

所以引入数据卷的目的就是为了解决上诉的问题，实现多个容器共享数据，保存数据、方便宿主机访问容器中的数据。数据卷是存在于一个或多个容器中的特定文件或文件夹，这个文件或文件夹以独立于 docker 文件系统的形式存在于宿主机中。

数据卷的最大特定是：==其生存周期独立于容器的生存周期。==

> 使用数据卷的最佳场景：
>
> ​	==在多个容器之间共享数据==，多个容器可以同时以只读或者读写的方式==挂载==(引用/指向的意思)同一个数据卷，从而共享数据卷中的数据。
>
> ​	当宿主机不能保证一定存在某个目录或一些固定路径的文件时，使用数据卷可以规避这种限制带来的问题。
>
> ​	当你想把容器中的数据存储在宿主机之外的地方时，比如远程主机上或云存储上。 
>
> ​	当你需要把==容器数据在不同的宿主机之间备份==、恢复或迁移时，数据卷是很好的选择。

#### 3.3 相关命令



```bat
#1 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认会存放到目录： /var/lib/docker/volume/数据卷名称/_data目录下
```

![image-20211104112451281](http://cdn.nefu-yzk.top/img/image-20211104112451281.png)



```bat
#2 查看数据卷
docker volume inspect 数据卷名称
#3 查看全部数据卷信息
docker volume ls
```

![image-20211104112546107](http://cdn.nefu-yzk.top/img/image-20211104112546107.png)



```bat
#4 删除数据卷
docker volume rm 数据卷名称
```



```bat
#5 应用数据卷
#5.1 当你映射数据卷时，如果数据卷不存在，Docker会帮你自动创建
# 即是将容器内的某个文件夹挂载到宿主机的指定文件夹中
docker run -v 数据卷名称:容器内路径 镜像ID
#当往数据卷中添加数据时，将同步至容器中，即指针指向了同一个文件夹

#5.2 直接指定一个路径作为数据卷的存储位置
docker run -v 路径:容器内部的路径 镜像ID

docker run --rm -d --name tomcat-8081 -p 8081:8080 -v /usr/local/docker/qfnj/:/usr/local/tomcat/webapps/qfnj tomcat

# v /usr/local/docker/qfnj/:/usr/local/tomcat/webapps/qfnj tomcat
# -v 数据卷参数。
# 将宿主机 /usr/local/docker/qfnj/ 文件内的内容信息 挂载在容器 /usr/local/tomcat/webapps/qfnj 目录下
```

![image-20211104113353676](http://cdn.nefu-yzk.top/img/image-20211104113353676.png)

使用cp命令朝容器的被数据卷指定的文件夹中放置文件时，文件也会同步到数据卷中去



## Centos防火墙端口

开放8080端口（如下命令只针对Centos7以上）

```bat
# 启动防火墙
sudo systemctl start firewalld
# 查看已经开放的端口
firewall-cmd --list-ports
```

![image-20211104113859291](http://cdn.nefu-yzk.top/img/image-20211104113859291.png)



```bat
# 开启某个端口
firewall-cmd --zone=public --add-port=8081/tcp --permanent
# 关闭某个端口
firewall-cmd --permanent --zone=public --remove-port=8081/tcp

# 注：开启和关闭某个端口都需要重启之后才能看到效果
```

![image-20211104114727129](http://cdn.nefu-yzk.top/img/image-20211104114727129.png)



```bat
firewall-cmd --reload #重启
firewall systemctl stop firewalld.service   #停止
firewall systemctl disable firewalld.service   #禁止firewall开机启动
```



## Docker安装开发环境

### 1、可能遇到的问题？

==问题描述：==一直卡在Pulling fs layer 不往下进行

![image-20211104143108766](http://cdn.nefu-yzk.top/img/image-20211104143108766.png)

==解决方式：==直接重启docke服务即可



==问题描述：==其他均下载完成，一直一层在waiting

![image-20211104153945658](http://cdn.nefu-yzk.top/img/image-20211104153945658.png)

==解决方式：==更换网络即可



```bat
# 获取nginx镜像
docker pull nginx  # 默认是latest版本

# 运行nginx容器
docker run -it --name nginx-80 -p 80:80 --rm -d nginx

# --name 容器的名称
# -p 80:80 端口进行映射，将本地的80端口映射到容器内部的80端口
# -d 后台运行该容器
# --rm 表示容器退出后直接删除该容器
# 上述运行后，直接可以访问nginx服务
```



![image-20211104154838244](http://cdn.nefu-yzk.top/img/image-20211104154838244.png)



```bat
# 创建目录
mkdir -p /usr/local/nginx

mkdir -p /usr/local/nginx/html
mkdir -p /usr/local/nginx/logs
mkdir -p /usr/local/nginx/conf

# 把容器内部的文件映射到所创建的目录中来
docker cp nginx-80:/etc/nginx/nginx.conf /usr/local/nginx/conf
docker cp nginx-80:/etc/nginx/conf.d /usr/local/nginx/conf

#启动tomcat
docker run --rm -d --name tomcat-8081 -p 8081:8080 -v /usr/local/docker/qfnj/:/usr/local/tomcat/webapps/qfnj tomcat

docker run --rm -d --name tomcat-8080 -p 8080:8080 -v /usr/local/docker/qfnj/:/usr/local/tomcat/webapps/qfnj tomcat

docker run --rm -d --name tomcat-8082 -p 8082:8080 -v /usr/local/docker/qfnj/:/usr/local/tomcat/webapps/qfnj tomcat

#启动nginx服务，并指定数据卷
docker run -it --name nginx-80 --rm -d -p 80:80 -v /usr/local/nginx/html:/usr/share/nginx/html -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/nginx/conf/conf.d/default.conf:/etc/nginx/conf.d/default.conf  -v /usr/local/nginx/logs:/var/log/nginx nginx

# 在html中随便建一个index.html文件即可访问
```

![image-20211104160639591](http://cdn.nefu-yzk.top/img/image-20211104160639591.png)

![image-20211104160603848](http://cdn.nefu-yzk.top/img/image-20211104160603848.png)



集群配置

```bat
# 第一步：修改nginx.conf文件
# vim /usr/local/nginx/conf/nginx.conf
    upstream nginxCluster{
        server 192.168.59.13:8080;
        server 192.168.59.13:8081;
        server 192.168.59.13:8082;
        }
        server {
                listen 80;
                server_name localhost;
                #charset koi8-r;
                #access_log /var/log/nginx/host.access.log main;
        location / { 
                proxy_pass http://nginxCluster;
                }
        }
# 第二步：修改conf.d/default.conf文件 
# vim /usr/local/nginx/conf/conf.d/default.conf
#添加该语句
location / {
                proxy_pass http://nginxCluster;
             }  
        }

```

![image-20211104162546833](http://cdn.nefu-yzk.top/img/image-20211104162546833.png)



![image-20211104162834009](http://cdn.nefu-yzk.top/img/image-20211104162834009.png)



### 2、安装MySQL

```bat
# 下载mysql镜像
docker pull mysql:5.6

# 创建并启动mysql
docker run -d --name mysql5.6-3306 -p 3306:3306 -e MYSQL_ROOT_PASSWORD='123456' mysql:5.6

#授权其他主机可以进行访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
# 然后刷新权限
FLUSH PRIVILEGES;
# 并退出
exit;
```



### 3、安装Redis

```bat
# 下载redis
docker pull redis:4.0.1

# 创建并运行redis容器
docker run --rm -d --name redis6379 -p 6379:6379 redis:4.0.1 --requirepass "123456"

# 进入redis容器内进行测试
docker exec -it redis6379 bash
redis-cli
```

## Docker定制镜像

> ​      镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题 就都会解决。这个脚本就是 Dockerfile。           
>
> ​       Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容， 就是描述该层应当如何构建



### 1、Dockerfile常用的命令

```bat
FROM   # 指定基础镜像，即后续的操作都是基于该镜像
MAINTAINER  # --提供Dockerfile 制作者提供本人信息
#LABLE --替代MAINTANIER 具体使用： LABLE maintainer="作者信息"
如例子所示
MAINTANIER "guoweixin <guoweixin@aliyun.com>"
LABEL maintainer="guoweixin@aliyun.com"
```



```bat
#ENV指令  
#   可以用于为docker容器设置环境变量 ENV设置的环境变量，可以使用 docker inspect命令来查看。同时 还可以使用docker run --env =来修改环境变量。
ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/:$JRE_HOME/lib/
ENV PATH $PATH:$JAVA_HOME/bin/

#workdir  用来切换工作目录的 默认是/
# WORKDIR 动作的目录改变是持久的，不用每个指令前都使用一次WORKDIR。

VOLUME  #创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等
# 只能定义docker管理的卷： VOLUME /data/mysql运行的时候会随机在宿主机的目录下生成一个卷目录！

COPY  # --把宿主机中的文件复制到镜像中去！


EXPOSE  # 为容器打开指定要监听的端口以实现与外部通信
# 使用格式： EXPOSE 80/tcp 23/udp
# 不加协议默认为tcp
# 使用-P选项可以暴露这里指定的端口！ 但是宿主的关联至这个端口的端口是随机的！

# 思考清楚每一层如何进行构建
```

==在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。== 



```bat
# 创建一个空白目录
mkdir -p /usr/local/docker/demo1
#创建一个文件
vim Dockerfile
#文件内容为：
FROM tomcat
RUN mkdir -p /usr/local/tomcat/webapps/ROOT/
RUN echo 'Hello qfnj Docker'>/usr/local/tomcat/webapps/ROOT/index.html

#构建镜像
docker build -t demo1 .
# -t 指定要创建的目标镜像明
# . Dockerfile文件所在的目录，可以指定Dockerfile的绝对路径

# 运行镜像所在的容器
docker run --rm --name demo1-8080 -p 8080:8080 -d demo1
```



### 2、注意点

RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这 样：

```bat
RUN apt-get update
RUN apt-get install -y gcc libc6-dev make
RUN wget http://download.redis.io/releases/redis-4.0.1.tar.gz
RUN tar xzf redis-4.0.1.tar.gz
RUN cd redis-4.0.1

```

> Dockerfile 中每一个指令都会建立一层，RUN 也不例外。每一个 RUN 的行为，和刚才我们手工建立镜像的过程一样： ==新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。== 而上面的这种写法，创建了多层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软 件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初 学 Docker 的人常犯的一个错误。

​        RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这 样：

```bat
FROM centos
RUN apt-get update \
			&& apt-get install -y gcc libc6-dev make \
			&& wget http://download.redis.io/releases/redis-			4.0.1.tar.gz \
			&& tar xzf redis-4.0.1.tar.gz \
			&& cd redis-4.0.1
```

> ​      首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的 事情。因此，这里没有使用很多个 RUN 对一一对应不同的命令，而是仅仅使用一个 RUN 指令，并使用 && 将各个所 需命令串联起来。将之前的 7 层，简化为了 1 层。==在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。== 并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 \ 的 命令换行方式，以及行首 # 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这 是一个比较好的习惯。



### 3、部署SpringBoot项目

准备好要部署的springboot项目

```bat
FROM java:8
volume /tmp
add seckill-demo.jar seckill.jar
expose 9090
entrypoint ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/seckill.jar"]

# FROM：表示基础镜像，即运行环境
# VOLUME /tmp创建/tmp目录并持久化到Docker数据文件夹，因为Spring Boot使用的内嵌Tomcat容器默认使
# 用/tmp作为工作目录
# ADD：拷贝文件并且重命名(ADD seckill-demo.jar seckill.jar 将应用jar包复制到/seckill.jar)
# EXPOSE：并不是真正的发布端口，这个只是容器部署人员与建立image的人员之间的交流，即建立image的人员告诉容器布署人员容器应该映射哪个端口给外界
# ENTRYPOINT：容器启动时运行的命令，相当于我们在命令行中输入java -jar xxxx.jar，为了缩短 Tomcat 的启动时间，添加java.security.egd的系统属性指向/dev/urandom作为 ENTRYPOINT


# 构建容器
docker build -t exam .

# 运行容器
docker run --rm -d --name 容器名称 -p 8080:8080 镜像名称
```



### 4、IDEA整合Docker



```bat
#修改该Docker服务文件
vi /lib/systemd/system/docker.service
#修改ExecStart这行
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

#重新加载配置文件
systemctl daemon-reload
#重启服务
systemctl restart docker.service
#查看端口是否开启
netstat -nlpt #如果找不到netstat命令，可进行安装。yum install net-tools
#直接curl看是否生效
curl http://127.0.0.1:2375/info

# 防火墙记得打开2375端口号，免得链接不上
firewall-cmd --zone=public --add-port=2375/tcp --permanent 
# 开启某个端口号之后，一定要重启防火墙才能让端口号生效
firewall-cmd --reload
```

![image-20211104195430404](http://cdn.nefu-yzk.top/img/image-20211104195430404.png)

![image-20211104195459384](http://cdn.nefu-yzk.top/img/image-20211104195459384.png)



==docker-maven-plugin==

> ​       传统过程中，打包、部署、等。 而在持续集成过程中，项目工程一般使用 Maven 编译打包，然后生成镜像，通过镜像上线，能够大大提供上线效率，同时能够快速动态扩容，快速回滚，着实很方便。==docker-maven-plugin== 插件就是为了帮助我们在Maven工程 中，通过简单的配置，自动生成镜像并推送到仓库中。



==pom.xml的完整文件==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.yzk</groupId>
    <artifactId>test</artifactId>
    <version>0.0.2</version>
    <name>test</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <!-- 镜像 前缀姓名-->
        <docker.image.prefix>yzk</docker.image.prefix>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
<!--                <version>1.0.0</version>-->
            </plugin>

            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <!-- 镜像名称 guoweixin/exam-->
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <!--指定标签，即版本号--> 
                    <imageTags>
                        <imageTag>0.0.4</imageTag>
                    </imageTags>
                    <!-- 基础镜像jdk 1.8-->
                    <baseImage>java</baseImage>
                    <!-- 制作者提供本人信息 -->
                    <maintainer>yzk 1586464596@qq.com</maintainer>
                    <!--切换到/ROOT目录 -->
                    <workdir>/ROOT</workdir>
                    <cmd>["java", "-version"]</cmd>
                    <entryPoint>["java", "-jar", "${project.build.finalName}.jar"]</entryPoint>
                    <!-- 指定 Dockerfile 路径
                    <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
                    -->
                    <!--指定远程 docker api地址，这是需要改成你对应的服务器地址-->
                    <dockerHost>http://192.168.59.13:2375</dockerHost>   
                    <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
                    <resources>
                        <resource>
                            <targetPath>/ROOT</targetPath>
                            <!--用于指定需要复制的根目录，${project.build.directory}表示target目录-->
                            <directory>${project.build.directory}</directory>
                            <!--用于指定需要复制的文件。${project.build.finalName}.jar指的是打包后的jar
                            包文件。-->
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>

                
                <!--当执行mvn package 时，执行： mvn clean package docker:build -->
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

如上用docker-maven插件 自动生成如下Dockerfile 文件：

```bat
FROM java
MAINTAINER yzk 1586464596@qq.com
WORKDIR /ROOT
ADD /ROOT/IDEA中的项目名.jar /ROOT/
ENTRYPOINT ["java", "-jar", "IDEA中的项目名.jar"]
CMD ["java", "-version"]
```

上述文件配置完成后，点击clean和package即可生成并推送该镜像到指定的服务器上

![image-20211105114047905](http://cdn.nefu-yzk.top/img/image-20211105114047905.png)



### 5、Idea整合Docker CA加密认证



```bat
# 1 创建ca文件夹，存放CA私钥和公钥
mkdir -p /usr/local/ca
cd /usr/local/ca

# 2 在Docker守护进程的主机上，生成CA私钥和公钥 并设置密码
openssl genrsa -aes256 -out ca-key.pem 4096

# 3 依次输入密码、国家、省、市、组织名称、邮箱等
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

# 4 生成server-key.pem
openssl genrsa -out server-key.pem 4096

# 5 CA来签署公钥 openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr  $Host换成你自己服务器外网的IP或者域名
openssl req -subj "/CN=192.168.59.13" -sha256 -new -key server-key.pem -out server.csr

# 6 配置白名单
# 允许指定ip可以连接到服务器的docker，可以配置ip，用逗号分隔开。
# 因为已经是ssl连接，所以我推荐配置0.0.0.0,也就是所有ip都可以连接(但只有拥有证书的才可以连接成功)，这样配置好之后公司其他人也可以使用。
# 如果填写的是ip地址 命令如下 echo subjectAltName = IP:$HOST,IP:0.0.0.0 >> extfile.cnf 
# 如果填写的是域名 命令如下 echo subjectAltName = DNS:$HOST,IP:0.0.0.0 >> extfile.cnf 
echo subjectAltName = IP:192.168.59.13,IP:0.0.0.0 >> extfile.cnf

# 7 执行命令 将Docker守护程序密钥的扩展使用属性设置为仅用于服务器身份验证
echo extendedKeyUsage = serverAuth >> extfile.cnf

# 8 生成签名证书
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

# 9 生成客户端的key.pem
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

# 10 要使密钥适合客户端身份验证 创建扩展配置文件
echo extendedKeyUsage = clientAuth >> extfile.cnf
echo extendedKeyUsage = clientAuth > extfile-client.cnf

# 11 生成签名证书 生成cert.pem,需要输入前面设置的密码
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf

# 12 删除不需要的文件，两个证书签名请求
# 生成cert.pem和server-cert之后。您可以安全地删除两个证书签名请求和扩展配置文件
rm -v client.csr server.csr extfile.cnf extfile-client.cnf

# 13 可修改权限
# 要保护您的密钥免受意外损坏，请删除其写入权限。要使它们只能被您读取，更改文件模式
chmod -v 0400 ca-key.pem key.pem server-key.pem
# 证书可以是对外可读的，删除写入权限以防止意外损坏
chmod -v 0444 ca.pem server-cert.pem cert.pem

# 14 归集服务器证书
cp server-*.pem /etc/docker/
cp ca.pem /etc/docker/

# 15 修改Docker配置 使Docker守护程序仅接受来自提供CA信任的证书的客户端的连接
vim /lib/systemd/system/docker.service
# 将ExecStart=/usr/bin/dockerd 替换为下面的东西：
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/usr/local/ca/ca.pem --tlscert=/usr/local/ca/server-cert.pem --tlskey=/usr/local/ca/server-key.pem -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

# 16 重新加载daemon并重启docker
systemctl daemon-reload
systemctl restart docker
# 17 开放2375端口
/sbin/iptables -I INPUT -p tcp --dport 2375 -j ACCEPT

# 18 重启 Docker
systemctl restart docker

# 上诉的18个步骤完成后，即可实现Docker的CA加密认证
```

#### 5.1 保存相关客户端的pem文件到本地

![image-20211105121112508](http://cdn.nefu-yzk.top/img/image-20211105121112508.png)

#### 5.2 IDEA CA配置

![image-20211105121152621](http://cdn.nefu-yzk.top/img/image-20211105121152621.png)


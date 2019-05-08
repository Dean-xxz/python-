## Docker学习笔记（一）
***
### 一、Docker简介
#### 1.Docker是什么
Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

#### 2.Docker应用场景
前三点在大多数应用开发过程中比较常用
  + web应用的自动化打包和发布
  + 自动化测试和持续集成、发布
  + 服务器环境中调整和部署数据库或其他应用
  + 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

#### 3.Docker优点
  + **简化程序**：
Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是 Docker的最大优势，过去需要用数天乃至数周的	任务，在Docker容器的处理下，只需要数秒就能完成。

  + **避免选择恐惧症**：
如果你有选择恐惧症，还是资深患者。Docker 帮你	打包你的纠结！比如 Docker 镜像；Docker 镜像中包含了运行环境和配置，所以 Docker 可以简化部署多种应用实例工作。比如 Web 应用、后台应用、数据库应用、大数据应用比如 Hadoop 集群、消息队列等等都可以打包成一个镜像部署。

  + **节省开支**：
一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker 改变了高性能必然高价格的思维定势。Docker 与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。
***
### 二、Docker架构
Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。
Docker 容器通过 Docker 镜像来创建。
容器与镜像的关系类似于面向对象编程中的对象与类。
  + Docker镜像（images）：创建Docker容器的模板
  + Docker容器（Container）：独立运行的一个或一组应用
  + Docker客户端（Client）：通过命令行或其他工具通过Docker API与Docker的守护进程尽心通信
  + Docker主机（Host）：一个物理或虚拟的机器用于执行守护进程和容器
  + Docker仓库（Registry）：Docker仓库用来保存镜像，类似于代码仓库，Docker Hub提供了很多镜像
  + Docker Machine：简化Docker安装的命令行工具
***

### 三、Docker安装
此处只讲centos系统安装
#### 1.版本支持
  + CentOS 7（64-bit）
  + CentOS 6.5 (64-bit)

#### 2.前提条件
  + 目前，CentOS 仅发行版本中的内核支持 Docker。
  + Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。
  + Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。
  + 查看系统版本
        uname -r

#### 3.使用yum安装docker
从 2017 年 3 月开始 docker 在原来的基础上分为两个分支版本: Docker CE 和 Docker EE。Docker CE 即社区免费版，Docker EE 即企业版，强调安全，但需付费使用。
  + 移除旧版本
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
  + 安装一些必要的系统工具
        sudo yum install -y yum-utils device-mapper-persistent-data lvm2
  + 添加软件源
        sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  + 更新 yum 缓存
        sudo yum makecache fast
  + 安装 Docker-ce
        sudo yum -y install docker-ce
  + 启动 Docker 后台服务
        sudo systemctl start docker
  + 测试运行 hello-world
        docker run hello-world

#### 4.使用脚本安装docker
  + 使用 sudo 或 root 权限登录 Centos
  + 确保 yum 包更新到最新
        sudo yum update
  + 执行 Docker 安装脚本
        curl -fsSL https://get.docker.com -o get-docker.sh
        sudo sh get-docker.sh
  + 启动 Docker 进程
        sudo systemctl start docker
  + 验证 docker 是否安装成功并在容器中执行一个测试的镜像
        sudo docker run hello-world
        docker ps

#### 5.镜像加速
鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是网易的镜像地址：http://hub-mirror.c.163.com，新版的 Docker 使用 /etc/docker/daemon.json来配置 Daemon请在该配置文件中加入（没有该文件的话，请先建一个）：

    {
      "registry-mirrors": ["http://hub-mirror.c.163.com"]
    }

#### 6.删除Docker
    sudo yum remove docker-ce
    sudo rm -rf /var/lib/docker
***
### 四、Docker使用
#### 1.Docker hello world
（1）Docker hello world

    docker run CentOS 7 /bin/echo "Hello world"
  + docker: Docker的二进制执行文件
  + run: 与前面的docker组合运行一个容器
  + conos7： 指定要运行的镜像，如果本地没有，就会从docker hub仓库拉取
  + /bin/echo "Hello world": 在启动的容器里执行的命令

（2）启动容器（后台模式）
  + 在输出中，我们没有看到期望的"hello world"，而是一串长字符，这个长字符串叫做容器ID，对每个容器来说都是唯一的
        docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1;done"
  + 确认容器是否在运行
        docker ps
  + 在容器内使用docker logs命令，查看容器内的标准输出
        docker logs 容器id/name


（3）停止容器

    docker stop 容器id/name

#### 2.Docker 容器使用(docker运行一个web应用)
（1）docker客户端
  + docker 客户端非常简单 ,我们可以直接输入 docker 命令来查看到 Docker 客户端的所有命令选项。
        docker
  + 可以通过命令 docker command --help 更深入的了解指定的 Docker 命令使用方法
        docker stats --help

（2）运行一个web应用
我们尝试使用 docker 构建一个 web 应用程序
        docker pull training/webapp  # 载入镜像
        docker run -d -P training/webapp python app.py
  + **-d**: 让容器在后台运行
  + **-P**：将容器内部使用的网络端口映射到我们使用的宿主机上

（3）查看web应用容器
  + 使用 docker ps 来查看我们正在运行的容器：
        docker ps
  + 通过访问主机对外端口，即可访问web应用
  + 我们也可以通过 -p 参数来设置不一样的端口：容器内部的 5000 端口映射到我们本地主机的 5000 端口上
        docker run -d -p 5000:5000 training/webapp python app.py

（4）网络端口的快捷方式
我可以使用 docker port 容器id 或 docker port name 来查看容器端口的映射情况。

    docker port bf08b7f2cd89
    docker port wizardly_chandrasekhar

（5）查看 WEB 应用程序日志
docker logs [ID或者名字] 可以查看容器内部的标准输出。

  docker logs -f bf08b7f2cd89
  + **-f**:让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出

（6）查看WEB应用程序容器的进程
我们还可以使用 docker top 来查看容器内部运行的进程

    docker top wizardly_chandrasekhar

（7）检查 WEB 应用程序
使用 docker inspect 来查看 Docker 的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。

    docker inspect wizardly_chandrasekhar

（8）停止 WEB 应用容器

    docker stop wizardly_chandrasekhar

（9）重启WEB应用容器
  + 已经停止的容器，我们可以使用命令 docker start 来启动。
        docker start wizardly_chandrasekhar
  + docker ps -l 查询最后一次创建的容器：
        docker ps -l
  + 正在运行的容器，我们可以使用 docker restart 命令来重启

（10）移除WEB应用容器

我们可以使用 docker rm 命令来删除不需要的容器,删除容器时，容器必须是停止状态

    docker rm wizardly_chandrasekhar


#### 3.Docker 镜像使用
（1）列出镜像列表
  + 使用 docker images 来列出本地主机上的镜像
        docker images
     * REPOSITORY：表示镜像的仓库源
     * TAG：镜像的标签（版本）
     * IMAGE ID：镜像id
     * CREATED：镜像创建时间
     * SIZE：镜像大小
  + 同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
  + 如果要使用版本为14.04的ubuntu系统镜像来运行容器时，命令如下：
        docker run -t -i ubuntu:14.04 /bin/bash

（2）获取一个新的镜像
  + 当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。
        docker pull ubuntu:13.10

（3）查找镜像
  + 我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： https://hub.docker.com/
  + 我们也可以使用 docker search 命令来搜索镜像。
        docker search httpd
    * NAME：镜像仓仓库源的名称
    * DESCRIPTION：镜像的描述
    * OFFICIAL：是否docker官方发布

（4）拖取镜像
  + 找到之后，就可以pull下载下来
        docker pull httpd
  + 然后直接使用了
        docker run httpd

（5）创建镜像

当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。
  + 从已经创建的容器中更新镜像，并且提交这个镜像
  + 使用 Dockerfile 指令来创建一个新的镜像

（6）更新镜像
  + 更新镜像之前，我们需要使用镜像来创建一个容器。
        docker run -t -i ubuntu:15.10 /bin/bash
  + 在运行的容器内使用 apt-get update 命令进行更新。
  + 在完成操作之后，输入 exit命令来退出这个容器。
  + 此时ID为e218edb10161的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit来提交容器副本。
        docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
  + 参数说明：
    * -m:提交的描述信息
    * -a:指定镜像作者
    * e218edb10161：容器ID
    * runoob/ubuntu:v2:指定要创建的目标镜像名

  + 我们可以使用 docker images 命令来查看我们的新镜像 runoob/ubuntu:v2：
  + 使用我们的新镜像 runoob/ubuntu 来启动一个容器
        docker run -t -i runoob/ubuntu:v2 /bin/bash

（7）构建镜像
  + 我们使用命令 docker build ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。
  runoob@runoob:~$ cat Dockerfile
        FROM    centos:6.7
        MAINTAINER      Fisher "fisher@sudops.com"

        RUN     /bin/echo 'root:123456' |chpasswd
        RUN     useradd runoob
        RUN     /bin/echo 'runoob:123456' |chpasswd
        RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
        EXPOSE  22
        EXPOSE  80
        CMD     /usr/sbin/sshd -D

     * 每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
     * 第一条FROM，指定使用哪个镜像源
     * RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。

  + 然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
        docker build -t runoob/centos:6.7 .
     * -t ：指定要创建的目标镜像名
     * . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

  + 可以使用docker images 查看新建的镜像

（8）设置镜像标签

我们可以使用 docker tag 命令，为镜像添加一个新的标签。

    docker tag 860c279d2fec runoob/centos:dev

docker tag 镜像ID，这里是 860c279d2fec ,用户名称、镜像源名(repository name)和新的标签名(tag)。
使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。

#### 4.Docker 容器连接
***
前面我们实现了通过网络端口来访问运行在 docker 容器内的服务。下面我们来实现通过端口连接到一个 docker 容器

（1）网络端口映射
  + 我们创建了一个 python 应用的容器。
        docker run -d -P training/webapp python app.py
  + 我们可以指定容器绑定的网络地址，比如绑定 127.0.0.1。我们使用 -P 参数创建一个容器，使用 docker ps 来看到端口 5000 绑定主机端口 32768,我们也可以使用 -p 标识来指定容器端口绑定到主机端口。
        docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py

     * -P :是容器内部端口随机映射到主机的高端口。
     * -p : 是容器内部端口绑定到指定的主机端口。
     * 这样我们就可以通过访问 127.0.0.1:5001 来访问容器的 5000 端口。

  + docker port 命令可以让我们快捷地查看端口的绑定情况。
        docker port adoring_stonebraker 5000

（2）docker容器连接
端口映射并不是唯一把 docker 连接到另一个容器的方法。docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。不是很懂，后面再讲！

（3）容器命名
当我们创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 --name 标识来命名容器，例如：

    docker run -d -P --name runoob training/webapp python app.py
***
### 五、Docker实例
这个模块后续会一一展开，敬请关注！
***
### 六、Docker参考手册
#### 1.docker命令大全
  + 容器生命周期管理
    * run：https://www.runoob.com/docker/docker-run-command.html
    * start/stop/restart: https://www.runoob.com/docker/docker-start-stop-restart-command.html
    * pause/unpause：https://www.runoob.com/docker/docker-kill-command.html
    * create：https://www.runoob.com/docker/docker-create-command.html
    * exec：https://www.runoob.com/docker/docker-exec-command.html

  + 容器操作
    * ps：https://www.runoob.com/docker/docker-ps-command.html
    * inspect：https://www.runoob.com/docker/docker-inspect-command.html
    * top：https://www.runoob.com/docker/docker-top-command.html
    * attach：https://www.runoob.com/docker/docker-attach-command.html
    * events：https://www.runoob.com/docker/docker-events-command.html
    * logs：https://www.runoob.com/docker/docker-logs-command.html
    * wait：https://www.runoob.com/docker/docker-wait-command.html
    * export：https://www.runoob.com/docker/docker-export-command.html
    * port：https://www.runoob.com/docker/docker-port-command.html

  + 容器rootfs命令
    * commit：https://www.runoob.com/docker/docker-commit-command.html
    * cp：https://www.runoob.com/docker/docker-cp-command.html
    * diff：https://www.runoob.com/docker/docker-diff-command.html

  + 镜像仓库
    * login：https://www.runoob.com/docker/docker-login-command.html
    * pull：https://www.runoob.com/docker/docker-pull-command.html
    * push：https://www.runoob.com/docker/docker-push-command.html
    * search：https://www.runoob.com/docker/docker-search-command.html

  + 本地镜像管理
    * images：https://www.runoob.com/docker/docker-images-command.html
    * rmi：https://www.runoob.com/docker/docker-rmi-command.html
    * tags：https://www.runoob.com/docker/docker-tag-command.html
    * build：https://www.runoob.com/docker/docker-build-command.html
    * history：https://www.runoob.com/docker/docker-history-command.html
    * save：https://www.runoob.com/docker/docker-save-command.html
    * load：https://www.runoob.com/docker/docker-load-command.html
    * import：https://www.runoob.com/docker/docker-import-command.html

  + info|version
    * info：https://www.runoob.com/docker/docker-info-command.html
    * version：https://www.runoob.com/docker/docker-version-command.html

#### 2.docker资源汇总
  + docker官方英文资源
    * docker官网：http://www.docker.com
    * Docker Windows 入门：https://docs.docker.com/docker-for-windows/
    * Docker CE(社区版) Ubuntu：https://docs.docker.com/install/linux/docker-ce/ubuntu/
    * Docker mac 入门：https://docs.docker.com/docker-for-mac/
    * Docker 用户指引：https://docs.docker.com/config/daemon/
    * Docker 官方博客：http://blog.docker.com/
    * Docker Hub: https://hub.docker.com/
    * Docker开源： https://www.docker.com/open-source

  + Docker中文资源
    * Docker中文网站：https://www.docker-cn.com/
    * Docker安装手册：https://docs.docker-cn.com/engine/installation/

  + Docker 国内镜像
    * 阿里云的加速器：https://help.aliyun.com/document_detail/60750.html
    * 网易加速器：http://hub-mirror.c.163.com
    * 官方中国加速器：https://registry.docker-cn.com
    * ustc的镜像：https://docker.mirrors.ustc.edu.cn

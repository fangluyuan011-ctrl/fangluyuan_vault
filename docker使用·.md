# Docker详细安装与使用教程：从入门到实践

## 引言        

​        Docker作为一种轻量级的容器化技术，以其隔离、便携、高效的特性，极大地简化了应用的部署、管理和扩展过程。本篇教程将带领您从零开始，逐步掌握Docker的安装、基础操作、镜像管理、容器运行以及网络与 数据卷设置，助您快速迈入Docker的世界。

## 一、Docker安装

1. ### 系统要求与准备
    
    ​    确保您的操作系统满足Docker的最低要求： 编程

 Linux：大多数主流Linux发行版（如Ubuntu、CentOS、Debian等）均支持Docker。确保内核版本高于3.10，且系统已更新至最新状态。
 macOS：安装Docker Desktop for Mac，要求macOS 10.14（Mojave）或更高版本。
Windows：对于Windows 10专业版、企业版或教育版，安装Docker Desktop；对于旧版Windows或家庭版，可使用Docker Toolbox。
2. ### Linux安装
    
    ​    使用包管理器安装
    ​    对于Ubuntu、Debian等基于Debian的系统：

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

 对于CentOS、RHEL等基于RPM的系统：

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
```


​        使用脚本安装
​        访问Docker官方安装脚本（https://get .docker.com），按照指示运行以自动安装。

3. ### macOS安装
    
    ​    访问Docker Desktop for Mac下载页面（https://www.docker.com/products/docker-desktop ），下载并安装最新版本的Docker Desktop。  
    
4. ### Windows安装
    
    ​    访问Docker Desktop for Windows 下载页面（https://www.docker.com/products/docker-desktop），下载并安装。对于不支持Docker Desktop的Windows版本，请参照官方文档安装Docker Toolbox。
    
5. ### 启动Docker服务
    
    ​    安装完成后，启动Docker服务：

```
sudo systemctl start docker
sudo systemctl enable docker # （Linux）设置开机自启动
```


或在macOS和Windows上启动Docker Desktop应用。 

## 二、Docker基础操作

1. 验证安装
        运行以下命令，如果输出Docker版本信息，表明安装成功：

```
docker --version
```

2. Hello World示例
        运行一个简单的Docker容器，验证Docker环境：

```
docker run hello-world
```

3. 基本命令一览
  docker images：列出本地镜像
  docker ps：列出正在运行的容器
  docker pull：下载镜像
  docker run：创建并启动容器
  docker stop：停止容器
  docker rm：删除容器
  docker rmi：删除镜像
  docker exec：在运行的容器中执行命令

  ## 三、镜像管理

  1.搜索镜像
      在Docker Hub或其他镜像仓库搜索镜像：

  ​    

  2.下载镜像
      下载指定镜像：

```
docker pull <image_name>:<tag>
```

  例如，下载官方的Ubuntu镜像： 开放源代码

```
docker pull ubuntu:latest
```



3. 构建镜像
        使用Dockerfile构建自定义镜像：

1.创建Dockerfile，编写构建指令。
2.在Dockerfile所在目录下，运行构建命令：

```
docker build -t <image_name>:<tag> .
```


4. 管理本地镜像
  列出本地镜像：docker images
  删除镜像：docker rmi <image_id_or_name>
  标签镜像：docker tag <image_id> <new_image_name>:<tag>

  ## 四、容器运行与管理

  1.创建并启动容器

  ```
  docker run [options] <image_name>:<tag> [command]
  ```


  常用选项包括：

-d：后台运行
-p：映射端口
-v：挂载 数据卷
--name：指定容器名
例如，启动一个交互式的Ubuntu容器：

```
docker run -it ubuntu:latest /bin/bash

```


2. 查看与管理容器
  列出容器：docker ps [-a]
  停止容器：docker stop <container_id_or_name>
  启动容器：docker start <container_id_or_name>
  重启容器：docker restart <container_id_or_name>
  进入容器：docker exec -it <container_id_or_name> bash
  查看容器日志：docker logs <container_id_or_name>

  ## 五、网络与数据卷

  1.网络
  创建网络：docker network create <network_name>
  连接容器到网络：docker run --network=<network_name> ...
  查看网络：docker network ls、docker network inspect <network_name>

  2.数据卷
  创建数据卷：docker volume create <volume_name>
  挂载数据卷到容器：docker run -v <volume_name>:<container_path> ...
  查看数据卷：docker volume ls、docker volume inspect <volume_name>

  ## 结语        

  ​     通过本教程，能帮助您大致掌握Docker的安装、基础操作、镜像管理、容器运行以及网络与数据卷设置。后续在使用Docker的过程中，建议持续探索更高级的主题，如Compose 文件、Swarm集群、Kubernetes集成等，以充分发挥Docker的潜力，提升开发与运维效率。
  ————————————————
  版权声明：本文为CSDN博主「进击的风筝」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/weixin_71699295/article/details/137387383
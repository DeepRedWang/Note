# Docker
## 安装Docker
#### 配置Docker服务
为避免每次使用docker命令均需要切换至特权身份，可将当前用户加入安装中自动创建的docker用户组
```bash
sudo usermod -aG docker user_name
eg: sudo usermod -aG docker wjy
#更新docker组
newgrp docker
```

## Docker镜像
#### 获取镜像docker pull
可从Docker Hub镜像源下载镜像REGISTRY为镜像仓库服务器，NAME为镜像仓库名称，TAG为镜像的标签(严格地讲，镜像仓库有名称中还应添加仓库地址，只是使用默认的Docker Hub服务，该前缀可以省略，若从非官方的仓库下载，则需要在仓库名称前指定完整的仓库地址；若不显示指定TAG，则会默认选择latest标签，会下载仓库中最新版本的镜像)
```bash
Docker pull [REGISTRY/NAME:TAG]
eg: docker pull registry.hub.docker.com/ubantu:18.04
```

#### 查看镜像信息docker images
docker images 或 docker image ls命令可以列出本地主机上已有镜像的基础信息
为了方便在后续工作中使用特定镜像，可以使用docker tag命令来为本地镜像任意添加新的标签
```bash
Docker tag [SOURSE_TAG] [TARGET_TAG]
eg: docker tag ubantu:latest myubantu:latest
```
添加tag后的镜像与原镜像的ID是完全一致的，它们实际上指向了同一个镜像，类似链接的作用。
###### 使用inspect命令来获取该镜像的详细信息

```bash
docker inspect ubantu:18.04
```
###### 使用history命令来查看镜像历史
```bash
docker history ubantu:18.04
```
过长的信息会被自动截断，可以使用--no-trunc来输出完整的命令，尤其是查看官方构成的时候非常有效；

#### 搜寻镜像
```bash
docker search [OPTINO] KEYWORD
eg: docker search --limit 10 pytorch
```

#### 清理和删除镜像
docker rmi IMAGE(orID)或 docker image rm IMAGE(orID)
当一个镜像拥有多个标签的时候(一个ID多个TAG)，该命令只是删除了该镜像多个标签中指定的标签，并不影响镜像文件。但只剩一个标签的时候就会彻底删除镜像。
若docker rmi 后面跟上镜像的ID，会先删除所有的标签，再删除镜像文件本身。但当有该镜像文件创建的容器存在时，镜像文件是无法被删除的。
若想强行删除镜像，使用-f参数，一般而言先删除镜像依赖的所有容器再删除镜像，删除容器命令 docker rm CONTAINER_ID
清理镜像，在使用docker一段时间后，系统遗留一些临时的镜像文件，以及一些没有被使用的镜像，使用如下命令来清除:
```bash
docker image prune 来清除
docker image prune -f (不需要任何提示)
```

#### 创建镜像
创建镜像有三种方法，基于已有的容器创建、本地模板导入、基于Dockerfile创建
###### 基于已有容器创建 
```bash
Docker commit [-OPTIONS] [CONTAINER_ID] [REPOSITORY:TAG]
eg: docker commit -m “Add a new file” -a “Deep Red” 12c810a08fd6 test:0.1
```
###### 基于dockerfile创建
待补充

## Docker容器
容器有三种状态，运行内状态，后台状态，停止状态
在容器内，处于运行状态
exit退出，容器处于停止状态(在运行时未--rm选项，若加了该选项，exit后会停止并删除容器)可通过docker ps -qa来查看停止状态的容器


#### 容器创建、启动、停止、删除
```bash
#创建
docker create CONTAINER_ID
#启动
docker start CONTAINER_ID
#创建并启动
docker run CONTAINER_ID

#停止容器
docker stop CONTAINER_ID

#删除容器
docker container rm CONTAINER_ID
#清理残留
docker container prune
```
|参数|功能|
|-|-|
|-it| 容器的标准输入打开(i)，分配伪终端(t)|
|-d |以后台形式运行|
|--rm| 退出容器后自动删除|
|--privileged| 以最高权限运行容器，不然在配置网络等操作会没有权限|

```bash
#后台运行容器并打开终端
docker -itd –runtime=nvidia [docker_images_name]
```

在容器中可以exit退出容器，此时容器处于停止状态
```bash
#重启处于终止状态的容器
docker start CONTAINER_ID
#再次进入容器,但这不是最好的办法，因为exit退出后还是退出了容器；
docker attach CONTAINER_ID
```

**更好的方式是exec，分配一个伪终端，即使退出伪终端也不会退出容器**
```bash
docker exec -it CONTAINER_ID /bin/bash
```

#### 查看容器运行状态及容器信息
```bash
#查看后台运行的容器
docker ps
#查看停止运行的容器
docker ps -qa

#查看容器信息
docker container inspect CONTAINER_ID
```

#### 导入导出容器
###### 导出容器
```bash
docker export -o FILENAME CONTAINER_ID
eg: docker  export  -o  my_ubuntu.tar  e3265
```
###### 导入容器
```bash
docker import FILENAME – REPOSITORY/IMAGE_NAME:TAG
eg: docker  import  my_ubuntu.tar  -  wangjianyang/ubuntu:0.1
```

## Docker Hub
#### 登录
docker login 
#### 搜索
docker search
#### 拉取
docker pull拉取，只有官方镜像可以不用指明仓库
若为Docker用户创建并维护的，带有用户名前缀，需要指定用户名
```bash
eg: docker pull wangjianyang/sae
```
若为第三方镜像市场，如腾讯云网易云等，需要在镜像仓库前指明注册服务器的地址
#### 上传
docker push
上传之前需要将docker镜像打标签，将自己的用户名作为仓库名



# docker使用

## 镜像
#### docker源设置
- 编辑/etc/docker/daemon.json
```
{
  "registry-mirrors" : [
    "http://ovfftd6p.mirror.aliyuncs.com",
    "http://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ],
  "insecure-registries" : [
    "registry.docker-cn.com",
    "docker.mirrors.ustc.edu.cn"
  ],
  "debug" : true,
  "experimental" : true
}
```
- 然后重启docker
#### 镜像搜索
```
docker image search
```
#### 镜像拉取
```
docker image pull 镜像名
```
#### 镜像列出
```
docker image ls
```
#### 镜像删除
```
docker image rm 镜像名
```
#### 将某个容器上传为镜像
```
docker commit 容器名 镜像名
```

## 容器
#### 容器创建
```
docker container create 参数
```
- 我个人经常使用的容器创建脚本命令，这样创建的容器既可以用命令行交互，也可使用图形界面
```
xhost +     #容器可使用宿主的GUI界面，因此宿主需打开xhost
docker container create \
        --name parsec-docker \
        -it \
        --ipc=host \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -e DISPLAY=$DISPLAY \
        -e QT_X11_NO_MITSHM=1 \
        --device=/dev/dri/card1:/dev/dri/card1 \    //显卡
        --device=/dev/snd:/dev/snd \                //声卡
        parsec-img \
        parsecd
  ```
#### 列出所有容器
 ```
 docker ps -a
 ```
#### 删除容器
 ```
 docker rm 容器名
 ```
#### 容器的开始，停止与重启
- 开始
 ```
 docker start 容器名
 ```
注：容器开始以后，既可以与其交互，用exec命令进入容器内, -it代表可交互
 ```
 docker exec -it 容器名 /bin/bash
 ```
 - 停止
 ```
 docker stop 容器名
 ```
- 重启
 ```
 docker restart 容器名
 ```
 
 ## 容器内存在普通用户，登录进容器的普通用户
 - 只需要在容器启动后，用exec进入时指定登录的用户即可，如下
  ```
  docker exec -it --user=用户名 容器名 /bin/bash
 ```
 
 ## 宿主创建好代理，如何让容器使用代理。我试过端口映射等方法都失败了，好像只有host模式成功了，如下
 - 创建容器时使用host模式
 ```
 --net=host
 ```
 - 设置容器的全局代理为与宿主代理一致，例如我现在宿主使用socks5代理，端口为1080，则容器内可以这样设置
 - 一，创建容器时使用-e设置全局变量
 ```
 -e ALL_PROXY="socks5://127.0.0.1:10808"    #因此是host模式，容器与宿主共用ip与端口，因此为127.0.0.1
 ```
 - 二，进入容器后使用export设置全局变量
 ```
 export ALL_PROXY="socks5://127.0.0.1:10808"
 ```
 - 三，当然linux本身支持的设置全局变量也可以，例如编辑/etc/profile


以上

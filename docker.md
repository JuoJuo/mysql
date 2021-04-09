### 进入容器

- docker container run centos 运行一个容器（容器跑的是对应的image）
- docker container ls 查看当前运行中的container  打印出来的数据，包含容器id，镜像名字，command表示运行容器后，立即执行的命令，还有创建时间，容器状态，端口映射情况，以及名字
- docker container ps 同上ls, -a所有 -l最近运行的
- docker container ls -a 查看所有的
- docker container run -it centos  /bin/bash 运行一个容器，不退出，分配了一个tts终端
- 进入一个正在运行的容器，docker container exec -it [containerId] /bin/bash
- docker container run -p 8080:80 nginx  运行一个容器，将宿主的8080与容器的80绑定 
<br>-p就是publish port的意思
<br>-e就是传递环境变量 eg：docker container run -p -e name="lrj" 8080:80 nginx
<br>这样写会在前台运行,不能敲别的命令.加个参数-d 就不会卡在那了。eg：docker container run -p -d 8080:80 nginx
<br>exit可以退出交互bash窗口
- docker container inspect [containerId] 可以查看容器的详细信息，包括环境变量，网络等信息
- docker logs [containerId] 查看容器的输出
- 停止、杀掉、启动、删除容器  docker container stop/kill/start/rm
- docker container commit -a "zuozheshilirenjie" -m "mynginxtest" [containerId]  [自己起名作为镜像名]:[自己起名作为版本号]，完了就去查看image，就有了。

> 还有很多container的命令，可以对container的各类配置进行修改，就不一一列举了。
> 镜像可以由docker build	根据Dockerfile构建出来，也可以由container commit出来

### 磁盘共享问题

###### volume 方式
在磁盘docker管理的文件夹路径里创建一个区域
docker volume create nginx-vol  
docker volume ls  
docker volume inspect nginx-vol #查看详情  
docker volume rm nginx-vol  删除volume  
docker volume ls -f dangling=true #列出已经孤立的数据盘  

#把nginx-vol数据卷挂载到/usr/share/nginx/html,挂载后容器内的文件会同步到数据卷中  
docker run -d  --name=nginx1 --mount src=nginx-vol,dst=/usr/share/nginx/html nginx  
简写：docker run -d  --name=nginx2  -v nginx-vol:/usr/share/nginx/html -p 3000:80 nginx  

######  Bind mounts 方式

docker run -v /mnt:/mnt -it --name logs centos bash 直接写磁盘上的一个位置，冒号容器内的地址，直接映射

###### 多个容器共享一个卷，继承的方式

docker create -v /mnt:/mnt --name logger centos 创建一个指定了mount目录的容器，但是不运行。  
docker run --volumes-from logger --name logger3 -i -t centos bash 使用刚创建的卷，运行容器。  
cd /mnt  
touch logger3  
docker run --volumes-from logger --name logger4 -i -t centos bash 使用刚创建的卷，运行容器。  
cd /mnt  
touch logger4  
两个container共享一个目录

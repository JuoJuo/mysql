
### 配置yum源
yum-config-manager --add-repo repository_url  
eg: yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo  
其实就是下载一个repo文件到/etc/yum.repos.d目录里边了  



### 安装无脑就行
yum install -y yum-utils   device-mapper-persistent-data   lvm2  
yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo  
yum install docker-ce docker-ce-cli containerd.io -y  

### image相关

- docker image ls会有4列，分别表示repository的名字(私人做的镜像一般名字带斜杠eg:lirenjie/nodejs,官方的不带)，tag表示版本，image id，创建时间，image的大小  
- docker search node 搜索带node关键字的所有镜像，输出5个列，镜像名、描述、stars数，是否是官方做的镜像，automated应该表示自己做的（仅仅推测）
- docker image(可选) history node 表示查看某个镜像之前执行了哪些命令.  --no-trunc表示不省略全显示(因为有时候输出太长了)
- docker image(可选) inspect node 查看镜像底层一点的信息，其中重要点的RootFS中的Layers比较重要，其实镜像是高度复用的，比如我使用官方的node镜像，在上面加了点东西，自己又做了一个新的镜像，这个layer会多一层的。一层一般就是一个功能，比方说node装在centos里的。那里面就有一个层是cenos。 层是高度复用的。
- docker image pull node
- docker image push xxx
- docker image rmi id or name
- docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]给本地的image起个别名，别名有规则的，REGISTRYHOST/imageName:tag  
eg: docker tag centos:7 zhangrenyang/centos:v1  
docker login 后直接可以用这个push，docker push zhangrenyang/centos:v1
- build	根据Dockerfile构建镜像  
docker build [OPTIONS] PATH | URL | -  
eg: path: docker build -t lirenjie/node:v1  
    url: docker build https://github.com/docker/rootfs.git#container:docker #后面表示分支:后面表示存储库中的一个子目录，该子目录将用作构建上下文。


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

### Dockerfile
```shell
#pull哪个镜像
FROM centos:6

# 多个lable就写多个 LABEL key2=value2
LABEL maintainer="SvenDowideit@home.org.au"

# 也可以这样： RUN ["executable", "param1", "param2"]
# 多个命令多写几个RUN或者CMD就好了，或者用&&连接多个命令
RUN yum install httpd  

# 也可以这样： CMD ["executable", "param1", "param2"]
# 多个命令多写几个RUN或者CMD就好了，或者用&&连接多个命令
# 这个是在镜像运行容器的时候执行
CMD /usr/sbin/sshd -D #

#容器内哪两个可以被外部发布出去（宿主端口还未指定）
EXPOSE 80 443

#设置环境变量 ENV <key>=<value> ...
ENV MYSQL_ROOT_PASSWORD=123456 test=123456

#给镜像添加文件进去，镜像里的目的地址可以是绝对路径，也可以是相对于WORKDIR的相对路径
ADD test.txt /absoluteDir/
# ADD支持url
ADD https://xxx.com/html.tar.gz /var/www.html /absoluteDir/
# ADD支持 自动解压压缩包(自动解压哈)
ADD nickdir.tar.gz ./

#仅允许您从主机（构建Docker映像的机器）的本地文件或目录中复制到Docker映像本身。 
COPY ./start.sh /start.sh

#目前认为跟CMD类似
# ENTRYPOINT ["executable", "param1", "param2"]
# ENTRYPOINT command param1 param2
ENTRYPOINT /bin/bash -c '/start.sh'

#只能指定镜像内的目录，宿主的目录没法在这个File中指定，随机分配。。。。。
VOLUME ["/var/lib/mysql"]

USER zhufengjiagou
# 整个image内的目录操作的，上下文目录
WORKDIR /data

# --build-arg <varname>=<value>构建iamge的时候可以传参哟！！！
# ARG variables are not persisted into the built image as ENV variables are
ARG user1=someuser
ARG buildno=1
```
### 网络
- 默认运行的容器就是桥接的，其他两种不常用
- docker network ls #列出当前的网络
- docker inspect bridge #查看当前的桥连网络  这样可以在多个container里ping通。
- docker run -d --name nginx1 nginx  因为是桥接的。但是每次启动可能ip就变了，我们就指定name属性，让容器在内部通过name或容器id能ping通。
- docker run -d --name nginx2 --link nginx1 nginx
```
docker exec -it nginx2 bash
apt update
apt install -y inetutils-ping  #ping
apt install -y dnsutils        #nslookup
apt install -y net-tools       #ifconfig
apt install -y iproute2        #ip
apt install -y curl            #curl
cat /etc/hosts
ping nginx1
```
#### 桥接有时候，我们还想搞多个桥接，每个网络互相独立，不能互通

# 创建自定义网络
docker network create --driver bridge dev_web
docker network create --driver bridge finalce_web
docker network ls

docker container run -d --name dev_nginx1 --net dev_web
docker container run -d --name dev_nginx2 --net dev_web

docker container run -d --name fin_nginx1 --net finalce_web
docker container run -d --name fin_nginx2 --net finalce_web

dev_web内的容器是互通的，但与finalce_web是不通的
同理finalce_web内的容器是互通的，但与dev_web是不通的
- 自定义网络自带DNS服务器，可以直接使用name或者容器id来ping，比之前省了--link
- 如果想dev_web与finalce_web相通，可以手动加
```
docker network connect dev_web fin_nginx1
docker network disconnect dev_web fin_nginx1
```

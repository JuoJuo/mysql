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

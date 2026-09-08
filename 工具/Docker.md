每个docker容器都是一个独立运行的运行环境（一个个独立的极简的Linux系统）
Cgroup限制每个容器的资源分配+Namespaces隔离进程的资源视图（容器只能看自身的进程及资源）

命令：
1：`docker pull docker.io/library/nginx:latest`
官方仓库/命名空间（作者名）/镜像:版本
简化：`docker pull nginx`

registry:仓库地址/注册表，如:`docker.io`
respsitory:镜像库（放同一个镜像的不同版本）:`docker pull docker.io/library/nginx(不带版本号)

`docker images`：列出下载的所有docker镜像
`docker rmi 镜像名\镜像id`:删除镜像；注意区分！`docker rm 容器名`：删除的是容器。如果容器正在运行还需加上`-f`强制删除。
`docker pull`还有个参数`--platform`：为镜像指定特定的cpu架构

`docker run 镜像名\ID`使用镜像创建并运行容器，
1.参数`-d`:分离模式，docker自动在后台运行、打印日志，不阻塞当前控制台；
容器内的网络与宿主机隔离，需要添加启动参数，将宿主机与容器的端口进行映射，
2.参数`-p num1:num2`：前方的是宿主机的端口，后者则是容器的端口。
3.参数`-v 宿主机目录：容器内目录`:宿主机的文件目录与容器内的目录做绑定，这样对宿主机进行文件的修改时，容器内也会响应，反之亦然，这称为“挂载卷”\绑定挂载。
4.参数`-e xxx`:向容器内传递环境变量。
5.参数`--name`:为容器命名
6.参数`-it 容器名`：让当前控制台进入容器
7.参数`--rm 容器名`：容器停止运行就删除，一般与6搭配使用，用于调试容器。
8.参数：`--restart 容器名`:配置容器在停.止时的重启策略：常用`--restart always 容器名`:宿主机一旦停止就重新启动；`--restart unless-stopped容器名 `：自动重启因意外原因而停止的容器，而不会重启被手动停止的容器。

查看容器日志：`docker logs 容器ID/名`（可再加`-f`:滚动查看日志，即追踪输出）

只想启停现有容器：
`docker start 容器ID/名`
`docker stop 容器ID/名`

只想先创建容器但不立即运行：`docker create 镜像名`，想启动接start命令。

查看容器详细信息：`docker inspect 容器名/ID`

卷：
1.让docker自动创建存储空间，为存储空间起名，挂载时直接使用该存储空间名字：命名卷挂载。
	创建挂载卷：`docker volume create 挂载卷名`;创建完成后直接可使用。
	查看挂载卷在宿主机的真实目录：`docker volume inspect`
命名卷第一次使用时docker会把容器文件夹同步到命名卷中，进行初始化，而绑定挂载无该功能。
2.`docker volume list`：查看当前所有卷
3.`docker volume rm 卷名`：删除卷
4.`docker volume prune -a`：删除没有任何容器在使用的卷

 `docker ps`:process status的缩写：查看正在运行的docker容器
可不pull直接run，因为run后如果docker发现本地没有该镜像，会自动拉取一份，然后再创建运行容器。`ps -a`查看所有容器（无论是否运行）


Docker的网络模式：
查看所有网络：`docker network list`
1.子网：`docker network create 子网名`，可让容器加入不同子网，同一子网内的容器可相互通信，不能跨子网通信，同一子网的容器可以直接使用容器名互相访问，不必使用IP地址。
创建容器时`--network 子网名`:指定容器加入的子网
docker子网内部有DNS机制，可把名字转换成IP地址
删除子网：`docker network rm 子网名

2.bridge桥接模式：宿主机与容器1配置了映射，此时假设还有一个容器2，没有与宿主机进行映射配置，但这个容器1与容器2在同一个子网中，此时尽管容器2没有与宿主机配置映射关系，宿主机仍可以访问到容器2.


2.Host模式：`--network host`
容器和宿主机共享网络，容器直接运行在宿主机的一个端口上，通过宿主机的IP地址以及端口就能访问到对应容器，无需进行端口映射

3.none模式（不联网）：



Linux相关：
`docker exec 容器ID\名 linux命令`：在容器内部执行linux命令
`docker exec -it 容器ID /bin/sh`:进入正在运行的容器中获得一个交互式命令行环境

Dockerfile:镜像的蓝图，使用Dockerfile来制作镜像，结构如下：
- FROM：选择一个的基础镜像，即我们的镜像是从哪个镜像的基础构建而来的。
- WORKDIR:切换到镜像内的一个目录，作为工作目录，后方命令均在此执行
- COPY：把我们的代码文件拷贝到镜像的工作目录，最起码写两个点：`. .`:第一个点：该电脑当前目录，第二个点：镜像内的当前工作目录
-  RUN 后跟命令：在镜像内执行的命令
- EXPOSE：声明镜像提供服务的端口
- CMD：容器运行时默认启动命令，每当容器启动时，容器内部会自动执行其后的命令。最好写成数组形式，一个Df文件仅能写一个CMD（与ENTRYPOINT类似，但EP优先级更高，不易被覆盖。）


Df文件写好后构建镜像：`docker build -t 给镜像的命名 （可写）:版本号 .(指文件构建的地方，.表示在当前文件夹构建) `

推送到docker hub：
1.docker login：登录
2.重新构建镜像，在镜像名前加用户名/
3.docker push 用户名/镜像名



多应用：（每个模块都打包成一个独立容器）
但开销大，难管理  ———》容器编排技术:docker compose 使用yml文件管理多个容器，其中列出容器如何创建以及协同应用，简单理解docker compose：一个或多个docker run命令，按特定格式列到一个文件里面。

docker会为每个compose文件自动创建一个子网，同个compose文件中定义的所有容器都将自动加入同一个子网

docker compose up
docker compose down：会停止并删除容器
docker compose stop：只停止不删除
docker compose start

docker compose只认准标准文件名，若使用非标准文件，需使用 -f 配置项进行指定
# Docker 命令

```bash
# 新版推荐使用 ls
docker container ls

# 所有镜像
docker container ls -a

# 老版本
docker container ps
docker container ps -a

# 停止某个镜像 CONTAINERID -> 镜像id
docker container stop CONTAINERID

# 删除某个镜像
docker container rm CONTAINERID

# 运行一个容器
docker container run nginx

# 端口映射 - 前台模式 --name webserver 自定义镜像名
docker container run -p 80:80 --name webserver nginx

# 端口映射 - 后台模式
docker container run -d -p 80:80 nginx

# 端口映射 - 前台转后台
docker attach CONTAINERID

# 查看已安装的镜像
docker images
```

[Docker run reference | Docker Documentation](https://docs.docker.com/engine/reference/run/)

## 帮助命令

```bash
# 显示 docker 版本信息
docker version

# docker 系统信息
docker info

# help
docker help
```



## 镜像命令

**`docker images` 查看本机所有镜像**

```shell
[root@VM-4-14-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    08b152afcfae   9 days ago     133MB
hello-world   latest    d1165f221234   4 months ago   13.3kB

# 解释
REPOSITORY	镜像的仓库源
TAG					镜像的标签
IMAGE ID		镜像ID
CREATED			镜像创建时间
SIZE				镜像大小

# 可选项
-a,  --all		# 列出所有镜像
-q,  --quiet	# 只显示镜像ID
```



**`docker search` 搜索镜像**

```bash
docker search --help


# 可选项 搜索 star 大于 5000 的镜像
docker search mysql -f STARS=5000

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
```



**`docker pull` 下载镜像**

```bash
# 下载 mysql
docker pull mysql

[root@VM-4-14-centos ~]# docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
33847f680f63: Already exists
5cb67864e624: Pull complete
1a2b594783f5: Pull complete
b30e406dd925: Pull complete
48901e306e4c: Pull complete
603d2b7147fd: Pull complete
802aa684c1c4: Pull complete
715d3c143a06: Pull complete
6978e1b7a511: Pull complete
f0d78b0ac1be: Pull complete
35a94d251ed1: Pull complete
36f75719b1a9: Pull complete
Digest: sha256:8b928a5117cf5c2238c7a09cd28c2e801ac98f91c3f8203a8938ae51f14700fd
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest

# 指定版本下载
docker pull mysql:5.7
```



**`docker remove` 删除镜像**

```bash
# 删除单个镜像
docker rmi -f d1165f221234

# 删除多个镜像
docker rmi -f id id id ...

# 删除所有镜像
docker rmi -f $(docker images -aq)
```





## 容器命令

> 有了镜像才可以创建容器



**启动命令简明版参数详解**

```bash
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]    
  
-d, --detach=false         指定容器运行于前台还是后台，默认为false     
-i, --interactive=false   打开STDIN，用于控制台交互    
-t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
-u, --user=""              指定容器的用户    
-a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
-w, --workdir=""           指定容器的工作目录   
-c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用    
-e, --env=[]               指定环境变量，容器中可以使用该环境变量    
-m, --memory=""            指定容器的内存上限    
-P, --publish-all=false    指定容器暴露的端口    
-p, --publish=[]           指定容器暴露的端口   
-h, --hostname=""          指定容器的主机名    
-v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
--volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
--cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
--cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
--cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
--cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
--device=[]                添加主机设备给容器，相当于设备直通    
--dns=[]                   指定容器的dns服务器    
--dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
--entrypoint=""            覆盖image的入口点    
--env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
--expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
--link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
--lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
--name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
--net="bridge"             容器网络设置:  
                           bridge 使用docker daemon指定的网桥       
                           host    //容器使用主机的网络    
            							 container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                           none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
--privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
--restart="no"             指定容器停止后的重启策略:  
                           no：容器退出时不重启    
                           on-failure：容器故障退出（返回值非零）时重启   
                           always：容器退出时总是重启    
--rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
--sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理    
```





**新建容器并启动**

```bash
docker run [可选参数] [镜像ID]

# 示例
docker run -d --name [nginx] -p 80:80 [镜像id]

# 参数说明
--name="NAME"		# 容器名称
-d							# 后台方式运行
-it							# 使用交互方式运行，进入容器查看内容
-p							# 指定容器端口 -p 80:80
	-p 主机端口:容器端口			# 常用
	-p 容器端口
	容器端口
-P							# 指定随机端口

# 启动，并进入 centos 容器
[root@VM-4-14-centos /]# docker run -it centos /bin/bash
[root@8c001fdb2ec9 /]# 

# 退出容器
[root@8c001fdb2ec9 /]# exit
```



**`docker ps` 查看正在运行的容器**

```bash
# 查看正在运行的容器
docker ps

-a 				# 查看历史运行的容器
-a -n=1		# 最近运行的 1 个容器
-aq 			# 列出所有容器 id
```



**退出容器**

```bash
# 退出容器并停止
exit

# 退出容器，不停止
Ctrl + P + Q

# 重新进入容器
docker attach [容器id]
```



**删除容器**

```bash
# 删除指定容器，不能删除正在运行的
docker rm [容器id]

# 删除所有容器
docker  rm -f $(docker ps -aq)

# 删除所有容器
docker ps -a -q|xargs docker rm
```



**启动、停止容器**

```bash
# 启动
docker start [容器id]

# 重启
docker restart [容器id]

# 停止
docker stop [容器id]

# 强制停止
docker kill [容器id]
```



**commit**

> 我们修改了容器的文件，也就是改动了容器的存储层。
>
> 可以通过 `docker diff` 查看具体改动

```bash
docker diff [容器id]
```

当我们运行一个容器的时候（如果不使用卷的话）。我们做任何文件修改都会被记录与容器存储层里。

而 docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加容器的存储层，并构成新的镜像。以后我们运行新镜像的时候，就会拥有原有容器最后的文件变化。

```bash
# 语法
docker commit [选项] <容器id或容器名> [<仓库名>:标签>]

# 示例
docker commit --author "Caeser <wangdaoo@yeah.net>" --message "修改了XXX" [容器id] nginx:v2

# 查看
docker images

# 查看历史提交
docker history nginx:v2

# 运行新镜像
docker run --name nginx-next -d -p 81:80 nginx:v2

# 查看
http://localhost:81
```



**⚠️慎用 docker commit**

使用 `docker commit` 命令虽然直观的帮助理解镜像分层存储的概念， **但！实际开发中并不会这样使用！**

首先，如果仔细观察之前的 `docker diff [容器名]` 的结果，你会发现除了真正修改的某个文件外，由于命令的执行，还有很多文件被改动或者添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那将会有大量的无关内容被添加或修改了。

此外，使用 `docker commit` 意味着所有对镜像的操作都是 **黑箱操作**，生成的镜像也是 **黑箱镜像**。换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是制作这个镜像的人，过一段时间后也无法记清具体的操作。这种黑箱镜像的维护工作是及其痛苦的。

而且，使用 `docker commit` 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。 **这会让镜像更加臃肿！**



## 常用命令

**查看 linux 运行状态**

```bash
docker stats
```



**后台启动**

```bash
# 后台启动
[root@VM-4-14-centos /]# docker run -d [容器id]

# docker ps, 发现 centos 没有运行
# 解释
# docker 容器使用后台运行，必须要有一个前台进程，docker 发现没有应用，就会立即停止
```



**查看日志**

```bash
# 显示日志
docker logs -tf [容器id]

# 显示的10条日志
docker logs -tf --tail 10 [容器id]
```



**查遍容器中的进程信息**

```bash
# 查看进程
docker top [容器id]
```



‼️**查看镜像的元数据** 

```bash
# 元数据
docker inspect [容器id]
```



‼️**进入当前正在运行的容器**

> 通常容器都在后台运行

```bash
# exec 方式进入
docker exec -it [容器id] /bin/bash
# 简写
docker exec -it [容器id] bash
# 进入容器后开启一个新的终端，可以在里面操作

# attach 方式进入
docker attach [容器id]
# 进入容器正在执行的终端，不会启动新的进程
```



**将容器内文件拷贝到主机**

```bash
# cp
docker cp [容器id]:[文件路径] [主机路径]
```

## 卷命令

**创建数据卷**

```bash
docker create [卷名称]
```



**查看数据卷**

```bash
docker inspect [卷名称]
```



**查看所有数据卷**

```bash
docker ls
```



**删除数据卷**

```bash
docker rm [卷名称]
```



**清理无用卷**

```bash
docker volume prune
```





## 小结

<img src="http://cdn.wangdaoo.com/image-20210801173222157.png" alt="image-20210801173222157" style="zoom:50%;" />

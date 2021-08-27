# Docker 容器数据卷



## 三种数据挂载方式

1. **volumes：** Docker管理宿主机文件系统的一部分  ‼️ **最常用的方式**
2. **bind mounts：** 意为着可以存储在宿主机系统的任意位置 🌛 **较常用**

> 但是，bind mount在不同的宿主机系统时不可移植的，比如Windows和Linux的目录结构是不一样的，bind mount所指向的host目录也不能一样。这也是为什么bind mount不能出现在Dockerfile中的原因，因为这样Dockerfile就不可移植了。

3. **tmpfs：** 挂载存储在宿主机系统的内存中，而不会写入宿主机的文件系统  ⚠️ **一般情况下不会使用**

<img src="http://cdn.wangdaoo.com/image-20210827100136654.png" alt="image-20210827100136654" style="zoom:50%;" />



## 什么是容器数据卷？

> 目录挂载！数据持久化和同步化！多个容器可以数据共享



## 位置

```bash
# 数据卷位置
cd /var/lib/docker/volumes
```

![image-20210827095110641](http://cdn.wangdaoo.com/image-20210827095110641.png)

> 由于没有在创建时指定卷，所以 Docker 帮我们默认创建了许多匿名卷





## 使用数据卷



**方式1：使用 -v 挂载**

```bash
# 进入容器
docker run -it -v /home/test:/home [镜像ID] bash

# 不进入容器
docker run -d -v /home/test:home [镜像ID]

# 查看容器详细信息
docker inspect [容器ID]
```

![image-20210820225319510](http://cdn.wangdaoo.com/image-20210820225319510.png)



**验证数据是否同步**

![image-20210820225956859](http://cdn.wangdaoo.com/image-20210820225956859.png)



## 实战 - 同步MySql



**思考：数据持久化**

```bash
# 搜索
docker search mysql

# 下载
docker pull mysql

# 启动容器
# -d 后台运行
# -p 端口映射
# -v 数据卷挂载
# -e 环境配置
# --name 容器名称
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name [容器名] [镜像ID]

# Navicat 测试连接
# 用户名：root
# 密码：123456
```



## 具名挂载 & 匿名挂载

```bash
# 匿名挂载
-v 容器内路径

# 具名挂载
-v 卷名:容器内路径

# 指定路径挂载
-v /宿主机路径:/容器内路径
```



## 拓展

```bash
# ro：readonly      - 只读
docker run -d -v /home/test:home:ro [镜像ID]

# rw: readwrite     - 可读可写
docker run -d -v /home/test:home:rw [镜像ID]
```


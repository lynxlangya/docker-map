# 容器数据卷 - VOLUME



## 什么是容器数据卷？

> 目录挂载！数据持久化和同步化！多个容器可以数据共享



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


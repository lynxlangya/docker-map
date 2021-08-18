# Docker 部署练习



## Nginx

```bash
# 搜索 -> 建议去 Docker Hub 上搜索，查看详细信息
docker search nginx

# 下载
docker pull nginx

# 查看镜像是否拉取
docker images

# 启动
docker run -d --name [自定义名称] -p 80:80 [镜像id]

# 查看容器是否启动
docker ps

# 访问
[服务器IP]:80

# 进入容器
docker attach [容器id]
# or
docker exec -it [容器id] /bin/bash

# 停止
docker stop [容器id]

# 重启
docker restart [容器id]
```



## Tomcat

```bash
# 搜索
docker search tomcat

# 下载
docker pull tomcat

# 查看镜像是否拉取
docker images

# 启动
docker run -d --name [自定义名称] -p 8080:8080 [镜像id]

# 访问服务器 8080 端口
404

# 进入容器内部
docker exec -it [容器id] /bin/bash

# 删除 webapps 文件
rm -rf webapps

# 重命名 webapps.dist
mv webapps.dist webapps

# 重新访问 - success
[服务器IP]:8080
```


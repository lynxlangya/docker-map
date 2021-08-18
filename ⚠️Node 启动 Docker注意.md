

# Node 启动 Docker注意⚠️

[docker-node/BestPractices.md at main · nodejs/docker-node (github.com)](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md#handling-kernel-signals)



**前端关注**

![image-20210804232347809](http://cdn.wangdaoo.com/image-20210804232347809.png)



‼️**为什么不能使用`CMD ['node', 'app.js']` 作为默认启动**



**这个问题涉及 `Linux` 运行机制，简单的说，就是 `Linux PI` 为 1 的进程是系统守护的进程，将会接受所有孤儿进程。并且在适当的时候发送关闭信号给这些进程**



**但是，`Docker` 中 `PID 1 `的进程为 `Node`，而 `Node` 并没有做回收孤儿进程的事情。所以，如果你的应用跑的类似爬虫之类的应用，执行完毕之后将进程挂到 `PID 1` 上，慢慢的容器就会 BOOM 💥**



## 解决



1. **用 `/bin/bash` 启动**
2. **在 `docker run ` 后面追加 `--init` 用于初始化一个 `docker` 进程为 `PID 1` 。docker 提供的进程可以回收所有孤儿进程**

```bash
docker run -it --init node
```



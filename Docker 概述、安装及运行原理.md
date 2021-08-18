# Docker

![image-20210803172128509](http://cdn.wangdaoo.com/image-20210803172128509.png)

> 弱小和物质不是生存的障碍，傲慢才是!

- Docker 概述
- Docker 安装
- Docker 命令
- Docker 镜像
- 容器数据卷
- DockerFile
- Docker 网络原理
- IDEA 整合 Docker
- Docker Compose (集群管理)
- Docker Swarm
- CI / CD （Jenkins）



## Docker 与 虚拟机（VM）

|    特性    |  Docker （容器）   | VM（虚拟机） |
| :--------: | :----------------: | :----------: |
|    启动    |        秒级        |    分钟级    |
|  硬盘使用  |     一般为 MB      |  一般为 GB   |
|    性能    |      接近原生      |     较弱     |
| 系统支持量 | 单机支持上千个容器 |    十来个    |

<img src="http://cdn.wangdaoo.com/image-20210804202449038.png" alt="image-20210804202449038" style="zoom:30%;" />



-----



<img src="http://cdn.wangdaoo.com/image-20210804202522836.png" alt="image-20210804202522836" style="zoom:20%;" />

**VM**

> VM 在宿主机器、宿主机器操作系统基础之上创建虚拟层，虚拟化操作系统，虚拟化仓库，然后安装应用



**Container**

> Docker 容器，在宿主机器、宿主机器操作系统之上创建 Docker 引擎，在引擎的基础上安装应用



## Docker 概述

**Docker 为什么会出现？**

产品上线：开发 -> 上线，两套环境

项目带着环境一起安装

[10分钟看懂Docker和K8S - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/53260098)



# Docker 核心

- 镜像 **Image**
- 容器 **Container**
- 仓库 **Repository**



**Docker 解决方案**

code -> build -> 打包带上环境（镜像）-> (Docker 仓库) -> 下载发布镜像 -> 直接运行即可

> Docker 思想来源于集装箱
>
> 隔离：`Docker` 核心思想，打包装箱，每个箱子互相隔离
>
> 隔离机制，将服务器利用到极致



## Docker 历史

2010 - `dotCloud`

2013 - `Docker` 开源

2014 - `Docker 1.0` 发布

`Docker` 十分轻巧！



## Docker 能做什么

**虚拟机技术缺点**

1. 资源占用十分多
2. 冗余步骤多
3. 启动很慢

![](http://cdn.wangdaoo.com/xuniji.png)

**容器化技术**

- 容器内的应用直接运行在宿主机内，容器没有自己的内核，也没有虚拟硬件
- 每个容器之间互相隔离，都有属于自己的文件系统

![](http://cdn.wangdaoo.com/rongqi.png)



> DevOps (开发、运维)

**应用更快速的交付和部署**

- 传统：一堆帮助文档，安装程序
- Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

- 使用 Docker 之后，部署应用和搭积木一样
- 项目打包未一个镜像，扩展 服务器A、服务器B

**更简单的系统运维**

- 容器化之后，我们开发、测试环境高度一致

**更高效的计算资源利用**

- Docker 是内核级别的虚拟化，可以在一个物理机上运行很多容器实例，服务器性能可以被压榨到极致



## Docker 基本组成



### 镜像（image）

![image-20210804223413996](http://cdn.wangdaoo.com/image-20210804223413996.png)

-----

![image-20210804223515626](http://cdn.wangdaoo.com/image-20210804223515626.png)

-----

![image-20210804223545685](http://cdn.wangdaoo.com/image-20210804223545685.png)

-----

‼️镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的配置参数（匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变

‼️镜像利用（`union file system`）提供应用运行的只读模版，它可以只提供一个功能，也可以由多个镜像叠加创建多个功能服务

‼️**Docker镜像是Docker容器的静态视角**



### 容器（container）

Docker 利用容器技术没独立运行一个或一组应用，通过镜像来创建

目前，可以把这个容器理解为一个简易的 Linux 系统



‼️镜像仅仅是定义隔离应用运行所需要的东西，容器则是运行这些镜像的进程。在容器内部，提供了完整的文件系统、网络、进程空间等。完全隔离外部环境，不会受到其他应用的侵扰。

‼️**容器**

容器的读写必须使用 `Volume` ，或者宿主的存储环境。因为容器在重启或关闭之后，存在与运行容器内部的数据都会丢失，每次启动容器，都是通过镜像创建一个新的容器

‼️**Docker容器是Docker镜像的运行状态**



### 仓库（repository）

‼️`Docker` 仓库是集中存放镜像文件的场地。镜像构建完成后，可以很容易的在当前宿主上运行。但是，如果需要在其他服务器上使用这个镜像，**我们就需要一个 集中的存储、分发镜像的服务。**

‼️`Docker Registry` （仓库注册服务器 `Docker Hub`）就是这样的服务。**有时候会把仓库（`Respository`）和仓库注册服务器（`Registry`）混为一谈，并不严格区分**。

‼️`Docker` 仓库的概念和 `Git` 类似，注册服务器可以理解为 `Github` 这样的托管服务。**一个 `Docker Registry` 中可以包含多个仓库，每个仓库可以包含多个标签（`Tag`），每个标签对应着一个镜像。** 所以说镜像仓库是 `Docker` 用来集中存放镜像文件的地方，类似于我们之前常用的代码仓库

‼️仓库又可以分为两种形式

- `public` 公有仓库
- `private` 私有仓库

![](http://cdn.wangdaoo.com/image-20210804225813540.png)



## Docker Client (Docker 客户端)

`Docker Client` 是一个泛称，用来向指定的 `Docker Engine` 发起请求，执行相应的容器管理操作。它既可以是 `Docker` 命令行工具，也可以是任何遵循了 `Docker API` 的客户端。



## Docker Engine (Docker 引擎)

`Docker Engine` 是 `Docker` 最核心的后台进程，它负责相应来自 `Docker Client` 的请求。然后将这些请求翻译成系统调用完成容器管理操作。该进程会在后台启动过一个 `API Server`，负责接收由 `Docker Client`  发送的请求，接收到的请求将通过 `Docker Engine` 内部的一个路由分发调度，再有具体的函数来执行请求





## 安装 Docker

### Linux 极简安装

```bash
curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker
```

> 此种安装方式会回根据不同操作系统安装体验版 （edge 版），而不是稳定版，最好不要用于生产环境

[史上最全（全平台）docker安装方法！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/54147784)



## Docker Run 流程

![image-20210801154529673](http://cdn.wangdaoo.com/image-20210801154529673.png)



## Docker 底层原理

**Docker 是如何工作的**

- Docker 是一个 Client - Server 结构的系统，Docker 的守护进程运行在主机上。通过 Socket 从客户端访问
- DockerServer 接收到 Docker - Client 的指令，就会执行这个命令

![image-20210801154818219](http://cdn.wangdaoo.com/image-20210801154818219.png)



**Docker 为什么比 VM 快？**

1. Docker 有着比 VM 更少的抽象层
2. Docker 利用的是宿主机的内核，VM需要 Guest OS

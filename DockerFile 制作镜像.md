# DockerFile 制作镜像

> 镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积问题就都会解决。 **这个脚本就是 Dockerfile**



Dockerfile 是一个文本文件，其内包含了一条条 **指令（Instruction）** ，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建



## 示例

**以定制 Nginx 为例**

在一个空白目录中，建立一个文本文件，命名为 `Dockerfile`

```bash
# 创建一个空文件夹
mkdir myNginx
# 进入
cd myNginx
# 创建 Dockerfile 文件
touch Dockerfile
```

**内容**

```bash
# Dockerfile 内容
FORM nginx
RUN echo '<h1>Hello, Dockerfile!</h1>' /usr/share/nginx/html/index.html
```





## 图示

![image-20210803175144248](http://cdn.wangdaoo.com/image-20210803175144248.png)

**举个例子**

```bash
# /usr/src/nodejs/hello-docker/Dockerfile
FROM node:10.0

# 维护者
MAINTAINER docker_user docker_user@email.com

# 在容器中创建一个目录
RUN mkdir -p /usr/src/nodejs/

# 定位到容器的工作目录
WORKDIR /usr/src/nodejs/

# RUN/COPY 是分层的，package.json 提前，只要没修改，就不会重新安装包
COPY package.json /usr/src/app/package.json
RUN cd /usr/src/app/
RUN npm i

# 把当前目录下的所有文件拷贝到 Image 的 /usr/src/nodejs/ 目录下
COPY . /usr/src/nodejs/

EXPOSE 3000
CMD npm start
```



## FORM

> FORM 是构建镜像的基础源镜像，该 Image 文件继承官方的 Image

DockerFile 中 FORM 是必备指令，且必须为第一条指令！它引入一个镜像作为我们要构建镜像的基础层。就好像我们要装操作系统，才可以在操作系统上安装、使用软件一样。

‼️ **`scratch`**  

> 除了选择现有镜像为基础镜像外，Docker 还存在一个特殊镜像，名为 `scratch` 。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像

```bash
FORM scratch
...
```

如果以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础。接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行的文件复制进镜像的做法并不罕见，对于 Linux 系统下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 `FORM scratch` 会让镜像体积更加小巧。使用 `GO 语言` 开发应用很多会使用这种方式来制作镜像，这也是为什么有人认为 `Go` 是特别适合容器微服务架构的语言的原因之一。



## **MAINTAINER**

> 维护者信息



## **RUN**

> `RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种



- `shell` ：`RUN <命令>` ，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 `RUN` 指令就是这种格式

```bash
RUN <命令>
RUN <命令>
...
```

❓既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 `Shell` 脚本一样把每个命令对应一个 `RUN` 呢？

**比如：**

```bash
# ❌错误写法
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```



Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样： **新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构建成新镜像**

而上面这种写法，创建了 7 层镜像。这是完全没意义的，而且很多运行时不需要的东西，都被装进来镜像了，比如编译环境、更新的软件包等等。结果就是产生非常臃肿的、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。

```bash
# ✅正确写法
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

首先，之前所有命令只有一个目的，就是编译、安装 `redis` 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 `RUN` ——对应不同的命令，而是仅仅用了一个 `RUN` 指令，并使用 `&&` 进行连接，将之前的 7 层，简化成 1 层。在撰写 Dockerfile 的时候，要经常提醒自己： **这并不是在写 `Shell` 脚本，而是定义每一层该如何构建**

并且，这里为了格式化还进行了换行。Dockerfile 支持 `Shell` 	类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个好习惯

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。这是很重要的一步，我们之前说过，镜像时多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加正在需要的东西，任何无关的东西都应该清理掉



每一个 `RUN` 指令都会新建立一层，在其上执行这些命令，频繁的使用 `RUN` 会创建大量的镜像层。然而 `Union FS` 是有最大指令层数限制的，不得超过 **127** 层，而且我们应该把每一层中没用的文件清除，比如一些没用的依赖，防止镜像臃肿。



## **WORKDIR**

> 容器的工作目录



## **COPY**

> 复制文件或目录



**格式**

- `COPY [--chown=<user>:<group>] <源路径>...<目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>"，..."<目标路径>"]`

`COPY` 指令将从构建上下文目录中`<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置

```bash
COPY package.json /usr/src/app/
```

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [filepath .Match](https://golang.org/pkg/path/filepath/#Match) 规则

```bash
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

`<目标路径>` 可是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建确实目录

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种源数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 `Git` 进行管理的时候

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组

```bash
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

**如果源路径为文件夹，复制的时候不是直接复制文件夹，而是将文件中的内容复制到目标路径**



## ADD 更高级的复制文件

> `ADD` 和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能

‼️在 Docker 官方的 [Dockerfile 最佳实践](https://yeasy.gitbook.io/docker_practice/appendix/best_practices) 中要求，尽可能的使用 `COPY` ，因为 `COPY` 的语义很明确，就是复制文件而已。而 `ADD` 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 `ADD` 的场合，就是所提及的需要自动解压缩的场合。

另外，需要注意的是，`ADD` 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢

‼️因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循

**所有的文件复制均使用 `COPY`**

**仅在需要自动解压缩的场合使用 `ADD`**

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组

```bash
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```



## CMD 容器启动命令

> `CMD` 指令的格式和 `RUN` 相似，也是两种格式

- `Shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ['可执行文件'，'参数1'，'参数2'...]`

> 参数列表格式：`CMD ['参数1'，'参数2'...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数



## ENTRYPOINT 入口点

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式



## ENV 设置环境变量

**格式**

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>`



## ARG 构建参数

**格式**

`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建构建环境的环境变量，在将来容器运行时是不会存在这些变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的

灵活使用 `ARG` 指令，能够在不修改 Dockerfile 的情况下，构建出不容的镜像

```bash
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```





## **EXPOSE**

> 设置容器服务端口



## **ENV**

> 设置环境变量



## VOLUME 定义匿名卷

**格式**

- `VOLUME ['<路径1>'，'<路径2>'...]`
- `VOLUME <路径>`

‼️ 容器运行时应该尽量保持容器存储层不发生变化写操作，对于数据库类需要保存动态数据的应用，其数据文件应该存储于卷 （`VLOUME`） 中，为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂在为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据

```bash
VOLUME /data
```

这里的 `/data` 目录就会在容器运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂在设置。

```bash
$ docker run -d -v mydata:/data xxxx
```

这行命令，就使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 `Dockerfile` 中定义的匿名卷的挂在配置



## USER 指定当前用户

**格式**

`USER <用户名>[:<用户组>]`



## HEALTHCHECK 健康检查

**格式**

- `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
- `HEALTHCHECK NODE`：如果基础镜像有健康检查指令，使用这行指令可以屏蔽掉其健康检查指令



## ONBUILD 为他人作嫁衣裳

**格式**

`ONBUILD <其他指令>` 



## LABEL 为镜像添加元数据

`LABEL` 指令用来给镜像以键值对的形式添加一些元数据 (metadata)

```bash
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```



## SHELL 指令

**格式**

`SHELL ['executable', 'parameters']`
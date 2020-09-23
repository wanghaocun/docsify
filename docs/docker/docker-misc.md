---
title: docker-misc
tags:
  - docker
  - tech
date: 2019-12-31 15:29:47
updated: 2019-12-31 16:29:47
categories: DDDD
permalink: 1577809488
index_img: https://filein.oss-cn-hangzhou.aliyuncs.com/chaos/one/docker-logo.png
banner_img: 
---



# Docker 札记

![docker-begin](https://filein.oss-cn-hangzhou.aliyuncs.com/chaos/one/docker-begin.png)

## 复制文件

```sh
# 复制docker文件到宿主机
docker cp mycontainer:/opt/file.txt /opt/local/
# 复制宿主机文件到docker
docker cp /opt/local/file.txt mycontainer:/opt/
```

## 帮助

> 想查看在创建容器时有哪些可用的选项，可以执行：

   ```sh 
docker run -it --rm image:tag --verbose --help
   ```

## 清理

> 清理所有停止运行的容器：

```sh
docker container prune
# or
docker rm $(docker ps -aq)
```

> 清理所有悬挂（``）镜像：

```bash
docker image prune
# or
docker rmi $(docker images -qf "dangling=true")
```

> 清理所有无用数据卷：

```shell
docker volume prune
```

由于`prune`操作是批量删除类的危险操作，所以会有一次确认。 如果不想输入`y`来确认，可以添加`-f`操作。慎用！

### 清理停止的容器

```sh
docker rm -lv CONTAINER
```

`-l`是清理link，`v`是清理volume。 这里的CONTAINER是容器的name或ID，可以是一个或多个。

参数列表：

| Name, shorthand | Default | Description                                             |
| --------------- | ------- | ------------------------------------------------------- |
| --force, -f     | false   | Force the removal of a running container (uses SIGKILL) |
| --link, -l      | false   | Remove the specified link                               |
| --volumes, -v   | false   | Remove the volumes associated with the container        |

### 清理所有停止的容器

通过`docker ps`可以查询当前运行的容器信息。 而通过`docker ps -a`，可以查询所有的容器信息，包括已停止的。

在需要清理所有已停止的容器时，通常利用shell的特性，组合一下就好。

```sh
docker rm $(docker ps -aq)
```

其中，`ps`的`-q`，是只输出容器ID，方便作为参数让`rm`使用。 假如给`rm`指定`-f`，则可以清理所有容器，包括正在运行的。

这条组合命令，等价于另一条命令：

```sh
docker container prune
```

`container`子命令，下面包含了所有和容器相关的子命令。 包括`docker ps`，等价于`docker container ps`或`docker container ls`。 其余还有`start`、`stop`、`kill`、`cp`等，一级子命令相当于二级子命令在外面的alias。 而`prune`则是特别提供的清理命令，这在其它的管理命令里还可以看到，比如`image`、`volume`。

### 按需批量清理容器

清除所有已停止的容器，是比较常用的清理。 但有时会需要做一些特殊过滤。

这时就需要使用`docker ps --filter`。

比如，显示所有返回值为0，即正常退出的容器：

```sh
docker ps -a --filter 'exited=0'
```

同理，可以得到其它非正常退出的容器。

目前支持的过滤器有：

> - id (container's id)
> - label (`label=` or `label==`)
> - name (container's name)
> - exited (int - the code of exited containers. Only useful with --all)
> - status (`created|restarting|running|removing|paused|exited|dead`)
> - ancestor (`[:]`, `` or ``) - filters containers that were created from the given image or a descendant.
> - before (container's id or name) - filters containers created before given id or name
> - since (container's id or name) - filters containers created since given id or name
> - isolation (`default|process|hyperv`) (Windows daemon only)
> - volume (volume name or mount point) - filters containers that mount volumes.
> - network (network id or name) - filters containers connected to the provided network
> - health (`starting|healthy|unhealthy|none`) - filters containers based on healthcheck status

### 清理失败

如果在清理容器时发生失败，通过重启Docker的Daemon，应该都能解决问题。

```sh
# systemd
sudo systemctl restart docker.service

# initd
sudo service docker restart
```

### 清理镜像

与清理容器的`ps`、`rm`类似，清理镜像也有`images`、`rmi`两个子命令。 `images`用来查看，`rmi`用来删除。

清理镜像前，应该确保该镜像的容器，已经被清除。

```sh
docker rmi IMAGE
```

其中，IMAGE可以是name或ID。 如果是name，不加TAG可以删除所有TAG。

另外，这两个命令也都属于alias。 `docker images`等价于`docker image ls`，而`docker rmi`等价于`docker image rm`。

### 按需批量清理镜像

与`ps`类似，`images`也支持`--filter`参数。

与清理相关，最常用的，当属``了。

```sh
docker images --filter "dangling=true"
```

这条命令，可以列出所有悬挂（dangling）的镜像，也就是显示为``的那些。

```sh
docker rmi $(docker images -qf "dangling=true")
```

这条组合命令，如果不写入Bash的alias，几乎无法使用。 不过还有一条等价命令，非常容易使用。

```sh
docker image prune
```

`prune`和`images`类似，也同样支持`--filter`参数。 其它的filter有：

> - dangling (boolean - true or false)
> - label (`label=` or `label==`)
> - before (`[:]`, `` or ``) - filter images created before given id or references
> - since (`[:]`, `` or ``) - filter images created since given id or references
> - reference (pattern of an image reference) - filter images whose reference matches the specified pattern

### 清理所有无用镜像

这招要慎用，否则需要重新下载。

```sh
docker image prune -a
```

### 清理数据卷

数据卷不如容器或镜像那样显眼，但占的硬盘却可大可小。

数据卷的相关命令，都在`docker volume`中了。

一般用`docker volume ls`来查看，用`docker volume rm VOLUME`来删除一个或多个。

不过，绝大多数情况下，不需要执行这两个命令的组合。 直接执行`docker volume prune`就好，即可删除所有无用卷。

注意：**这是一个危险操作！甚至可以说，这是本文中最危险的操作！** 一般真正有价值的运行数据，都在数据卷中。 （当然也可能挂载到了容器外的文件系统里，那就没关系。） 如果在关键服务停止期间，执行这个操作，很可能会**丢失所有数据**！

### 从文件系统删除

除配置文件以为，Docker的内容相关文件，基本都放在`/var/lib/docker/`目录下。

该目录下有下列子目录，基本可以猜测出用途：

> - aufs
> - containers
> - image
> - network
> - plugins
> - swarm
> - tmp
> - trust
> - volumes

一般不推荐直接操作这些目录，除非一些极特殊情况。 操作不当，后果难料，需要慎重。

## `docker save` 和 `docker load`

Docker 还提供了 `docker save` 和 `docker load` 命令，用以将镜像保存为一个文件，然后传输到另一个位置上，再加载进来。这是在没有 Docker Registry 时的做法，现在已经不推荐，镜像迁移应该直接使用 Docker Registry，无论是直接使用 Docker Hub 还是使用内网私有 Registry 都可以。

### 保存镜像

使用 `docker save` 命令可以将镜像保存为归档文件。

比如我们希望保存这个 `alpine` 镜像。

```bash
$ docker image ls alpine
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              baa5d63471ea        5 weeks ago         4.803 MB
```

保存镜像的命令为：

```bash
$ docker save alpine -o filename
$ file filename
filename: POSIX tar archive
```

这里的 filename 可以为任意名称甚至任意后缀名，但文件的本质都是归档文件

**注意：如果同名则会覆盖（没有警告）**

若使用 `gzip` 压缩：

```bash
$ docker save alpine | gzip > alpine-latest.tar.gz
```

然后我们将 `alpine-latest.tar.gz` 文件复制到了到了另一个机器上，可以用下面这个命令加载镜像：

```bash
$ docker load -i alpine-latest.tar.gz
Loaded image: alpine:latest
```

如果我们结合这两个命令以及 `ssh` 甚至 `pv` 的话，利用 Linux 强大的管道，我们可以写一个命令完成从一个机器将镜像迁移到另一个机器，并且带进度条的功能：

```bash
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```


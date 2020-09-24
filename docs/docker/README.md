# docker memo
[toc]

## time zone issue
参考笔记 [《Docker 时区调整方案》](http://note.youdao.com/noteshare?id=7f541ca3b715591c2d76e7a96325009f&sub=C739A09EFFA746B2AE360F27EC5140A1)
- Dockerfile构建重写
```dockerfile
...
RUN mkdir -p /some/path \
    && chmod 777 /some/path \
    ==============================================================
    && rm -rf /etc/localtime \
    && /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' >/etc/timezone
    ==============================================================
...
```
- run 加环境变量 -e TZ（并非所有容器都生效）
```docker
docker run ... \
=====================
-e TZ=Asia/Shanghai \
=====================
...
```

## manual
### docker_practice
[online](https://vuepress.mirror.docker-practice.com/)  
[offline](https://github.com/yeasy/docker_practice/wiki/%E7%A6%BB%E7%BA%BF%E9%98%85%E8%AF%BB%E5%8A%9F%E8%83%BD%E8%AF%A6%E8%A7%A3)
```docker
# 为了保持内容为最新，建议每次阅读前先 pull 最新镜像

# GitBook 格式
docker pull ccr.ccs.tencentyun.com/dockerpracticesig/docker_practice
docker run -d -p 14000:80 --name docker_practice_gitbook ccr.ccs.tencentyun.com/dockerpracticesig/docker_practice

# Vuepress 格式
docker pull ccr.ccs.tencentyun.com/dockerpracticesig/docker_practice:vuepress
docker run -d -p 24000:80 --name docker_practice_vuepress  ccr.ccs.tencentyun.com/dockerpracticesig/docker_practice:vuepress
```

### docker/getting-started
```docker
docker pull docker/getting-started
docker run -d -p 34000:80 --name docker-getting-started docker/getting-started
```

## repo
- [docker hub](https://hub.docker.com/)  
- [daocloud-道客](https://hub.daocloud.io/repos)  
- [网易云](https://c.163yun.com/hub#/home)

## mirror
[docker-practice](https://github.com/yeasy/docker_practice/blob/master/install/mirror.md)
> [mirror-test](https://github.com/docker-practice/docker-registry-cn-mirror-test/actions)

- [阿里云镜像源](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) `需要登录`
- [科大镜像源](https://mirrors.ustc.edu.cn/help/dockerhub.html) [`文档`](https://lug.ustc.edu.cn/wiki/mirrors/help/docker)
- [道客镜像源](https://www.daocloud.io/mirror#accelerator-doc) [`文档`](http://guide.daocloud.io/dcs/docker-9153151.html)
- [七牛云镜像源](https://kirk-enterprise.github.io/hub-docs/#/user-guide/mirror)
- [网易云镜像源](https://www.163yun.com/help/documents/56918246390157312)
- [百度云镜像源](https://cloud.baidu.com/doc/CCE/s/Yjxppt74z#%E4%BD%BF%E7%94%A8dockerhub%E5%8A%A0%E9%80%9F%E5%99%A8)

```json
{
  "registry-mirrors": [
    "https://gsidpfxa.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn/",
    "http://f1361db2.m.daocloud.io",
    "https://reg-mirror.qiniu.com",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

## mysql
> 基于`Debian`构建  
[docker hub info](https://hub.docker.com/_/mysql)  
[custom.cnf](http://note.youdao.com/noteshare?id=909efdec4e372b8119ad834c73924a17)
### 8(latest)

```bash
# Windows cmd
docker run -d -p 3306:3306 \
-v a:/Docker/mysql/conf/custom.cnf:/etc/mysql/conf.d/custom.cnf \
-v a:/Docker/mysql/data:/var/lib/mysql \
-v a:/Docker/mysql/logs:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-e TZ=Asia/Shanghai \
--security-opt seccomp=unconfined \
--name mysql --network dev --restart=always \
mysql

# MySQL8中默认字符集[--character-set-server=utf8mb4]
# 默认排序规则[--collation-server=utf8mb4_0900_ai_ci]
# [-e TZ=Asia/Shanghai] -- custom.cnf文件的时区设置修改不了日志文件使用的时区
# [--security-opt seccomp=unconfined] -- 不显示报错信息(mbind: Operation not permitted)（不设置也不影响使用）


# 校验参数
SELECT NOW();
show variables like 'slow_query%';
show variables like 'long_query_time';
show variables like 'slow_query_log_file';

SELECT @@global.time_zone;
```

> // 暂未发现下面问题，暂不修改  
> MySQL8中，默认排序规则居然从utf8mb4_general_ci修改为了utf8mb4_0900_ai_ci,造成某些特殊字符插入不进去，这里把MySQL的默认排序规则重新修改了utf8mb4_general_ci  
>
> >collation-server=utf8mb4_general_ci 

> mbind: Operation not permitted问题 -> [seccomp](https://docs.docker.com/engine/security/seccomp/)

> [Docker MySQL 8 慢查询日志监控详解](https://blog.csdn.net/zgpeace/article/details/104114000)

> 报错信息[全局可写]`mysqld: [Warning] World-writable config file '/etc/mysql/conf.d/custom.cnf' is ignored.` -> [sof-link](https://stackoverflow.com/questions/37001272/fixing-world-writable-mysql-error-in-docker) `Windows简单将文件改为只读属性即可；Linux使用chmod修改.cnf文件权限(755)`

> [日志时区问题](https://stackoverflow.com/questions/35123049/how-to-change-time-zone-of-error-log-file-of-mysql)

cnf-file:  
***[Manual](https://dev.mysql.com/doc/refman/8.0/en/)***
> [server-system-variables](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)  
> [mysql-command-options](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html)  
> [mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)  
> [mysqlbinlog](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html)  
> [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)  
> [mysqldumpslow](https://dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html)

```
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
# 使用了[-e TZ=Asia/Shanghai]永久设置系统时区
#default_time_zone=+8:00

general-log=ON
general-log-file=/var/log/mysql/mysql.log

log-error=/var/log/mysql/error.log

slow_query_log =ON
long_query_time=9
slow_query_log_file=/var/log/mysql/slowquery.log
#log-queries-not-using-indexes=ON

#lower_case_table_names=1

# 解决日志文件中的时区问题
log_timestamps = SYSTEM
```

### 5.7
```docker
docker run -d -p 3306:3306  \
-v a:/Docker/mysql/conf/custom.cnf:/etc/mysql/conf.d/custom.cnf \
-v a:/Docker/mysql/data:/var/lib/mysql \
-v a:/Docker/mysql/logs:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-e TZ=Asia/Shanghai \
--name mysql5 \
--network dev \
--restart=always \
mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

## redis
> 基于`Debian`构建  
[docker hub info](https://hub.docker.com/_/redis)  
[redis.conf](http://note.youdao.com/noteshare?id=16d350cf644fb17b31599b7ce12ca207)

```docker
# 带验证简单启动 启动aof
docker run -d --name redisdock -p 16379:6379 redis --requirepass "123456" --appendonly yes

# 挂载conf并持久化启动
# window10
docker run -d -p 6379:6379 -v /a/docker/redis/conf/redis.conf:/etc/redis/redis.conf -v /a/docker/redis/data:/data -e TZ=Asia/Shanghai --name redis --network dev --restart=always redis redis-server /etc/redis/redis.conf

# centos7
docker run -d -p 26379:6379 \
-v /usr/local/docker/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /usr/local/docker/redis/data:/data \
-e TZ=Asia/Shanghai \
--name hanlp-api-hub-redis6 --network dev --restart=always \
redis:latest redis-server /etc/redis/redis.conf
```

> 手动设置密码

```docker
# 进入redis的容器
docker exec -it redis /bin/bash
#  运行命令
redis-cli

# 查看现有的redis密码
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
# 设置redis密码
127.0.0.1:6379>config set requirepass ****（****为你要设置的密码）
OK
```

**如何查看已运行的容器的docker run启动参数**
```docker
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock assaflavie/runlike redis(查看其它的，更换对应的container名称即可)
```
> [redis conf](https://redis.io/topics/config)  
> [my conf](http://note.youdao.com/noteshare?id=16d350cf644fb17b31599b7ce12ca207)
>
>  常用参数：
>```pseudocode
>  # 网络开放
>  bind 0.0.0.0
>  # 端口号
>  port 6379
>  # 安全验证
>  requirepass wanghaocun
>  # RDB同步时间
>  save 900 1
>  save 300 10
>  save 60 10000
>  # RDB文件路径
>  dir ./
>  # 开启AOF
>  appendonly yes
>  # AOF同步时间
>  appendfsync everysec
>  # RDB & AOF混合持久化  
>  aof-use-rdb-preamble yes
>```
>
> [redis persistence](https://redis.io/topics/persistence)  
> [混合持久化](https://blog.csdn.net/yhl_jxy/article/details/91879874)

## mongo
> 基于`Ubuntu`构建  
[docker hub info](https://hub.docker.com/_/mongo)
```docker
docker network create dev

# 简单启动mongo服务实例
docker run -d -p 27017:27017 -v /a/Docker/mongo/data:/data/db -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=123456 --name mongo --network dev --restart=always  mongo
```
> Navicat连接后记得勾选`查看`->`显示隐藏的项目`菜单项，否则默认数据库不显示

## mongo-express
[docker hub info](https://hub.docker.com/_/mongo-express)
```docker
# 简单启动mongo-express服务实例
docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=root -e ME_CONFIG_MONGODB_ADMINPASSWORD=123456 -e ME_CONFIG_BASICAUTH_USERNAME=admin -e ME_CONFIG_BASICAUTH_PASSWORD=123456 -e ME_CONFIG_OPTIONS_EDITORTHEME=monokai --name mongo-express --network dev --restart=always mongo-express
```

## rabbitmq
> 基于`Ubuntu`构建  
[admin-guide](https://www.rabbitmq.com/admin-guide.html)
```bash
# [rabbitmq:3-management]带web管理
# 制定USER/PASS不会创建guest用户
# -e TZ="Asia/Shanghai" 不生效
docker run -d -p 5672:5672 -p 15672:15672 \
-h dev-rabbit \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=123456 \
--name rabbit \
--network dev \
rabbitmq:3-management

docker exec -it rabbitmq /bin/bash

# 服务器状态
rabbitmqctl status

# 用户管理
rabbitmqctl list_users
rabbitmqctl add_user mall mall
rabbitmqctl delete_user mall
rabbitmqctl change_password mall 123456
# tag可以为administrator, monitoring, management
rabbitmqctl set_user_tags mall administrator

# 权限管理
rabbitmqctl list_permissions
rabbitmqctl list_user_permissions mall
rabbitmqctl clear_permissions [-p /mall] mall
rabbitmqctl set_permissions -p /mall mall “.*” “.*” “.*”

# vhost管理
rabbitmqctl add_vhost /mall
rabbitmqctl delete_vhost /mall
```

user tags
```pseudocode
Comma-separated list of tags to apply to the user. Currently supported by the management plugin:

management
User can access the management plugin
policymaker
User can access the management plugin and manage policies and parameters for the vhosts they have access to.
monitoring
User can access the management plugin and see all connections and channels as well as node-related information.
administrator
User can do everything monitoring can do, manage users, vhosts and permissions, close other user's connections, and manage policies and parameters for all vhosts.

Note that you can set any tag here; the links for the above four tags are just for convenience.
```

## ELK
> Note: Pulling an images requires using a specific version number tag. The latest tag is not supported.


### elasticsearch
> 基于`CentOS`构建  
[docker hub info](https://hub.docker.com/_/elasticsearch)

```docker
# 创建容器内桥接网络用于kibana等
docker network create dev
# 运行容器
docker run -d -p 9200:9200 -p 9300:9300 \
-e discovery.type=single-node \
-e cluster.name=elasticsearch \
-e TZ=Asia/Shanghai \
-v a:/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v a:/docker/elasticsearch/data:/usr/share/elasticsearch/data \
--name elasticsearch --network dev --restart=always \
elasticsearch:7.6.2

docker exec -it elasticsearch /bin/bash

elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip

docker restart elasticsearch
```


### logstash
> 基于`CentOS`构建  
[docker hub info](https://hub.docker.com/_/logstash)

```docker
docker run -d -p 4560:4560 -p 4561:4561 -p 4562:4562 -p 4563:4563 \
-v a:/docker/logstash/conf/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-e TZ=Asia/Shanghai \
--name logstash --network dev --restart=always \
logstash:7.6.2

# 进入容器内部，安装`json_lines`插件
logstash-plugin install logstash-codec-json_lines
```


### kibana
> 基于`CentOS`构建  
[docker hub info](https://hub.docker.com/_/kibana)  

```docker
# 使用es网络内部连接
docker run -d -p 5601:5601 \
-e TZ=Asia/Shanghai \
--name kibana --network dev --restart=always \
kibana:7.6.2
```

> Note: In this example, Kibana is using the default configuration and expects to connect to a running Elasticsearch instance at http://localhost:9200

> Kibana can be accessed by browser via http://localhost:5601 or http://host-ip:5601




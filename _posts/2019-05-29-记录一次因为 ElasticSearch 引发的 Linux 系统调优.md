---
layout:     post
title:      一次因为 ElasticSearch 引发的系统调优
subtitle:   事务基础
date:       2019-03-11
author:     Alessio
header-img: img/PostBack_03.jpg
catalog: true
tags:
    - ElasticSearch
    - SpringBoot
    - java
---

## 前倾概要

最近需要为手头的项目引入自己的搜索引擎，很自然地选择了 ElasticSearch，在经理技术选型之后，确定当前版本如下：
- maven 依赖
```xml
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-elasticsearch</artifactId>
        <version>3.0.14.RELEASE</version>
    </dependency>
```

- 在服务部署方面选择 docker ，以下是组件版本

```bash
kibana                    5                   62079cf74c23        2 months ago        396MB
elasticsearch             5.6.12              de05e10fa879        7 months ago        486MB

```
## 调优引发

在进行部署的时候，我选择使用外部配置文件覆盖容器内配置文件的方式，准备好了如下的配置文件，并在容器创建的时候进行指定

- 配置文件
```bash
┌[root@localhost] [/dev/pts/1] 
└[~]> cat /usr/share/elasticsearch.yml 
http.host: 0.0.0.0
# Uncomment the following lines for a production cluster deployment
transport.host: 0.0.0.0
#discovery.zen.minimum_master_nodes: 1
```
开启 `transport.host` 配置行是为了让我的 java 应用能够访问到 9300 端口，在此也可指定具体地址，向我一样放开，则是“来者不拒”的配置方式
- 启动命令

```bash
docker run -di --name=tensquare_elasticsearch -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 -v /usr/share/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml de05e10fa879

```
在命令中我制定了端口映射规则和服务启动方式 `single-node` ，在启动之后，使用 `docker ps` 命令查看当前 docker进程，但是并没有找到对应的容器——很明显，容器挂了

## 调优过程

在进行依赖冲突排查和服务部署排查之后，有对当前情况做了相近调查搜索，我确定了当前问题的根源——**并没有足够的资源支持这个容器启动**，这就意味着我们有必要修改 Linux 的系统配置，来做一次调优

### 修改 `/etc/security/limits.conf` 文件

```bash
#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4
*		soft	nofile		65536
*		hard	nofile		65536
# End of file
```
在这个文件中我们放大软硬件资源限制

`nofile` 是单个进程允许打开的最大文件个数

`soft nofile` 是软限制 

`hard nofile` 是硬限制

### 修改 `/etc/sysctl.conf` 文件
在文件最后追加这行配置
```bash
vm.max_map_count=655360
```
这行配置可以限制一个进程可以拥有的 `VMA` ( 虚拟内存区域 ) 的数量

### 调优生效

执行命令，使调优立即生效
```bash
sysctl ‐p
```

### 结束语

调优结束，重启 docker 容器，现在你的容器可以快活的生存啦
```bash
root@localhost: ~ # docker container ls                                                                                                                             
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                            NAME
7ea81893596a        de05e10fa879              "/docker-entrypoint.…"   8 minutes ago       Up 19 seconds       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   search
root@localhost: ~ # curl http://localhost:9200                                                                                                                      
{
  "name" : "64suvuC",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "5W1AwZb4T0iZJiqFOWolFg",
  "version" : {
    "number" : "5.6.12",
    "build_hash" : "cfe3d9f",
    "build_date" : "2018-09-10T20:12:43.732Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```
---
layout:     post
title:      文章拿在手 Redis 面试不用愁（二）
subtitle:   Redis 高可用之主从复制和Sentinel哨兵模式
date:       2019-04-19
author:     Alessio
header-img: img/PostBack_10.jpg
catalog: true
tags:
    - Redis
---
## 从 Redis 主从复制开始谈

复制作为我们在开发和日常运维中的重要一环，在提供服务高可用方面创造了非常良好的环境。

Redis 中为我们提供了非常方便的复制方案。体现在配置文件上，我们只需要一条配置就可以完成复制的开启

我们在这里即将谈到的主从复制，就是这个方案

### 主从复制配置文件详解

我们从配置文件中，把关于主从复制的配置行摘取部分：

```bash
replica-serve-stale-data yes
replica-read-only yes
replica-priority 100
replicaof <masterip> <masterport>
# min-replicas-to-write 3
# min-replicas-max-lag 10
# replica-announce-ip 5.5.5.5
# replica-announce-port 1234
replica-lazy-flush no
# cluster-replica-no-failover no
client-output-buffer-limit replica 256mb 64mb 60
```
我们只要修改 `replicaof <masterip> <masterport>` 配置行，为当前节点指定主节点。即可开启默认的主从复制配置

### 主从复制原理

#### 主从复制的流程
简单来说，主从复制的详细流程可以分为如下阶段：

1. 设置主节点信息
2. 建立连接并验证
3. 交换信息并同步
4. 命令广播

我们一步一步地解释这个流程

- 设置主节点信息

我们在修改上述配置行后，将会为两台 Redis 节点之间建立主从联系。这就是设置主从信息的步骤了。

说的形象一点，就是为主节点指派了一个 “秘书”。这个秘书会忠实地记录在主节点上的所有数据。

这个秘书在默认配置下不对外提供写入功能，只是作为主节点的记录着和服务的 “后备军” 存在。像开启从节点的写入，我们需要修改配置文件中的 `replica-read-only yes` 配置行。

对应的，这个“秘书”也会详细地保存主节点的信息，我们可以在开启主从复制后，查看 `info replicaton` 信息项，即可检查当前节点的主从信息，在这里将列出一个 **一主一从** 配置的节点信息。

**主节点信息查看**

```bash
-!- /developer/rediscfg » redis-cli -p 6380 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=210,lag=1
master_replid:df113c994a038314c1427b481f4e1c9277c454a8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:210
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:210
```

**从节点信息查看**

```bash
-!- /developer/rediscfg » redis-cli -p 6381 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:14
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:df113c994a038314c1427b481f4e1c9277c454a8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
```
我们在主从信息统计里面尤其要注意 `master_repl_offset` 这个信息。但是这个信息为什么重要，请允许我在这里卖个关子，我们很快就会说到它。

- 建立连接并验证

在设置好主从信息之后，从节点将会向主节点建立 socket 链接，成功后将会发送 `ping` 命令给主节点。

`ping` 命令将起到检查整个链接通路的作用，检查 socket 读写状态是否正常，检查主节点能否处理命令请求。

在收到主节点的 `pong` 回复之后，从节点将会检查 `masterauth` 配置行，进行身份验证。如果此配置行未设置，将会默认跳过检查，但是如果身份验证未通过，将会不断执行重试，并打印出日志

- 交换信息并同步

在身份验证通过之后，从节点将会向主节点发送自己的监听端口信息，做好同步准备。主节点则接收从节点信息并记录在 `info replication` 信息段中，准备向从节点同步数据。

同步信息是由从节点开始的。从节点在信息交换完成后会向主节点发送 `PSYNC` 命令（在 Redis2.8 版本之前是 `SYNC`），开始同步。

##### 全量同步和部分重同步

这是 Redis 采用的两种同步策略，可以简单的理解为 “全部同步” 和 “缺省部分同步”

## 高可用之 Sentinel 模式

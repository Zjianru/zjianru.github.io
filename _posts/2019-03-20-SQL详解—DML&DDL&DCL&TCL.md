---
layout:     post
title:      SQL 详解
subtitle:   DML & DDL & DCL & TCL
date:       2019-03-20
author:     Alessio
header-img: img/PostBack_03.jpg
catalog: true
tags:
    - SQL
---
**DML（Data Manipulation Language）数据操纵语言**

由 DBMS 提供 实现对数据库中数据的操作。

分类

-   交互型 DML

-   嵌入型 DML

依据语言的级别 又可分

-   过程性 DML

-   非过程性 DML

需要commit 但某些数据库做了默认提交处理

-   `SELECT`

    -   见 QL 篇

-   `INSERT`

    -   `INSERT INTO table(conlumns) VALUES(conlumns)`

    -   `INSERT INTO table ( ) SELECT`

-  ` UPDATE table SET col = val ,col = val`

-   `DELETE table WHERE`

-  `MERGE`

-   `CALL`

-   `EXPLAIN PLAN`

-   `LOCK TABLE`

 

**DDL（Data Definition Language）数据库定义语言**

用于定义数据库的三级结构，包括外模式、概念模式、内模式及其相互之间的映像，定义数据的完整性、安全控制等约束

-   `CREATE`

-   `ALTER` 修改表结构

-   `DROP` 删除

-   `TRUNCATE table` 高端指针复原 重新设置索引 不可被回滚

-   `COMMENT`

-   `RENAME`

 

**DCL**（**Data Control Language**）**数据库控制语言** 

-   `GRANT` 授权

>   举例并详解

```sql
grant all privileges on *.* to 'user'@'%' identified by 'passwd' with grant option;
```


-   `all privileges`

    -   将所有权限授予给后面指定的用户

    -   也可指定具体的权限 如 SELECT CREATE DROP 等

-   `on`

    -   指定权限将对数据和表生效

    -   格式： `Database.Table`

    -   这里的 `*` 表示所有数据库 所有表

-   `to`

    -   指定被授予权限的用户

    -   格式：“user”@“IP”

-   `identified by`

    -   指定用户登陆密码

-   `with grant option`

    -   表示允许用户将自己的权限授予给其他用户

>    

-   `REVOKE` 取消授权

>   举例并解释

```sql
revoke all on test.* from user@'localhost';
```
-   `all` 权限

-   `on` 收回用户对哪个库哪张表的权限 这里是test库的所有表

    -   可用 \* 表示全部

-   `from` 收回哪个用户的权限

    -   后接用户名和 ip (登陆方式)

>    

**TCL（Transaction Control Language）事务控制语言**

`SAVEPOINT` 设置保存点

`ROLLBACK ` 回滚

`SET TRANSACTION`
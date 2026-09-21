---
title: OracleRAC集群打开归档日志
date: 2023-12-08 06:28:15
updated: 2026-08-20 13:14:03
description: Oracle RAC 集群开启归档模式的停库、参数修改、启动与验证流程。
categories:
  - Oracle 数据库
tags:
  - Oracle
cover: /img/covers/enable-oracle-rac-archivelog.png
top_img: /img/covers/enable-oracle-rac-archivelog.png
comments: false
index: true
source: https://blog.xiaopangza.cn/archives/enable-oracle-rac-archivelog
---

> 本文同步自 [Halo 主博客](https://blog.xiaopangza.cn/archives/enable-oracle-rac-archivelog)，内容以两站最新版本为准。
## 一·oracle用户查看归档日志状态

### 1.查看归档状态

```shell
[oracle@rac1 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Fri Aug 11 12:17:37 2023

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,
Data Mining and Real Application Testing options

SQL> archive log list;
Database log mode	       Archive Mode --归档模式 --No Archive Mode 非归档模式
Automatic archival	       Enabled --自动归档 开启
Archive destination	       USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     505058
Next log sequence to archive   505059
Current log sequence	       505059
SQL>

```

### 2.节点实例状态：

```shell
SQL> show parameter cluster

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
cluster_database		     boolean	 TRUE
cluster_database_instances	     integer	 2
cluster_interconnects		     string
SQL> select instance_name,host_name,status from gv$instance;

INSTANCE_NAME
----------------
HOST_NAME							 STATUS
---------------------------------------------------------------- ------------
xjxsora1
rac1								 OPEN

xjxsora2
rac2								 OPEN


SQL>
```

实例两节点都为开启状态

### 3.数据库集群参数：

```shell
SQL> show parameter cluster

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
cluster_database		     boolean	 TRUE
cluster_database_instances	     integer	 2
cluster_interconnects		     string
SQL>
```

cluster_database为true表示为集群数据库，否则，非集群数据库

## 二.开启数据库归档模式

***注: 以下操作只在一个节点上执行，本次在节点1(xjxsora1)进行的操作**

### 1.备份参数文件：

```shell
SQL> create pfile='/u01/app/oraInventory/backup/spfilebak.ora' from spfile;
```

本次将文件备份至*/u01/app/oraInventory/backup/*目录下的*spfilebak.ora*

### 2.修改cluster_database参数:

修改为非集群数据库，该参数为静态参数，需要使用scope=spfile

```shell
SQL> alter system  set cluster_database=false scope=spfile sid='*';
```

### 3.切换到grid用户

新建一个shell连接切换至grid用户

```shell
[root@rac1 ~]# su - grid
上一次登录：五 8月 11 11:38:41 CST 2023pts/3 上
[grid@rac1 ~]$
```

在grid用户下***停止***数据库：

```shell
[grid@rac1 ~] $ srvctl stop database -d xjxsora     ------将数据库一致停库
[grid@rac1 ~] $ srvctl start instance -d xjxsora -i xjxsora1 -o mount                -------将节点1启动到mount状态
```

### 4.节点1切换到oracle用户登录数据库中：

查询数据库实例状态：

```shell
[oracle@rac1 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Fri Aug 11 12:17:37 2023

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,
Data Mining and Real Application Testing options

SQL> select instance_name,status from v$instance;

         INSTANCE_NAME    STATUS
         ---------------- ------------
         xjxsora1          MOUNTED
SQL>
```

修改数据库成归档模式：

```shell
SQL> alter database archivelog;
     Database altered.
SQL>
```

将集群参数修改回去：

```shell
SQL> alter system set cluster_database=true scope=spfile sid='*';
   System altered.
SQL>
```

### 5.切换到grid用户，关闭整个数据库，然后重启：

```shell
[grid@rac1 ~]$ srvctl stop database -d xjxsora
[grid@rac1 ~]$ srvctl start database -d xjxsora
```

### 6.切换到oracle用户下登录数据库查询归档状态：

```shell
SQL> archive log list;
Database log mode	       Archive Mode --归档模式
Automatic archival	       Enabled --自动归档开启
Archive destination	       USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     505058
Next log sequence to archive   505059
Current log sequence	       505059
SQL>
```

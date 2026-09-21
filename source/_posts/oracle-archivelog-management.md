---
title: Oracle归档日志管理
date: 2023-12-08 06:28:00
updated: 2026-08-20 13:13:59
description: Oracle 归档日志空间检查、RMAN 清理、脚本维护与定时任务配置。
categories:
  - Oracle 数据库
tags:
  - Oracle
cover: /img/covers/oracle-archivelog-management.png
top_img: /img/covers/oracle-archivelog-management.png
comments: false
index: true
source: https://blog.xiaopangza.cn/archives/oracle-archivelog-management
---

> 本文同步自 [Halo 主博客](https://blog.xiaopangza.cn/archives/oracle-archivelog-management)，内容以两站最新版本为准。
如果Oracle启用了日志归档模式，必须保证有足够的磁盘空间存放归档日志文件，如果空间不足，在线日志不能归档，也不会切换，数据库将暂停运行，错误代码为*ORA-00257*。

所以，归档日志文件要定期清理，而由于当前项目归档日志过大，故于每天一点清理前一天的归档日志

## 一.查看归档日志

### 1.查看归档日志占比

oracle用户登录或root用户登录后切换成oracle用户

```bash
[root@rac1 ~]# su - oracle 上一次登录：五 8月 11 11:09:31 CST 2023从 11.64.3.52pts/4 上 [oracle@rac1 ~]$
```

进入oracle数据库

```bash
[oracle@rac1 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Fri Aug 11 13:19:20 2023

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,
Data Mining and Real Application Testing options

SQL>

```

执行 select * from V$FLASH_RECOVERY_AREA_USAGE;命令查看日志大小(百分比显示)

```bash
SQL> select * from V$FLASH_RECOVERY_AREA_USAGE;

FILE_TYPE	     PERCENT_SPACE_USED PERCENT_SPACE_RECLAIMABLE
-------------------- ------------------ -------------------------
NUMBER_OF_FILES
---------------
CONTROL FILE			    .02 			0
	      1

REDO LOG			     .2 			0
	      4

ARCHIVED LOG			  78.11 			0
	  12328


FILE_TYPE	     PERCENT_SPACE_USED PERCENT_SPACE_RECLAIMABLE
-------------------- ------------------ -------------------------
NUMBER_OF_FILES
---------------
BACKUP PIECE			      0 			0
	      0

IMAGE COPY			      0 			0
	      0

FLASHBACK LOG			      0 			0
	      0


FILE_TYPE	     PERCENT_SPACE_USED PERCENT_SPACE_RECLAIMABLE
-------------------- ------------------ -------------------------
NUMBER_OF_FILES
---------------
FOREIGN ARCHIVED LOG		      0 			0
	      0


7 rows selected.

SQL>
```

原则上超过80%就需要清理，超过90%就有宕机风险

### 2.查看归档日志目录

```bash
SQL> show parameter db_recovery_file_dest;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest		     string	 +DATA
db_recovery_file_dest_size	     big integer 250G
SQL>

```

最大大小是250G

## 二.解决归档日志问题

### 1.增大归档日志空间

oracle用户登录或root用户登录后切换成oracle用户

```bash
[root@rac1 ~]# su - oracle
上一次登录：五 8月 11 11:09:31 CST 2023从 11.64.3.52pts/4 上
[oracle@rac1 ~]$
```

进入oracle数据库

```bash
[oracle@rac1 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Fri Aug 11 13:19:20 2023

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, Real Application Clusters, Automatic Storage Management, OLAP,
Data Mining and Real Application Testing options

SQL>

```

增加归档日志所在空间大小 注：不能超过剩余磁盘空间

```bash
SQL> alter system set db_recovery_file_dest_size=500G;
System altered.
```

### 2.清理归档日志

#### 2.1 oracle用户打开RMAN工具清理

```bash
[oracle@rac1 app]$ rman target /

Recovery Manager: Release 11.2.0.4.0 - Production on Fri Aug 11 13:49:44 2023

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

connected to target database: XJXSORA (DBID=1154684129)

RMAN>

```

检查一些无用的归档日志

```bash
RMAN> crosscheck archivelog all;
```

删除过期的归档,会删除所有归档日志

```bash
RMAN> delete expired archivelog all;
```

保留日志删除

sysdate - x x为几就保留几天

```bash
RMAN> delete archivelog until time 'sysdate - 1';
```

#### 2.2 oracle用户使用脚本清理

在rac1上已经创建了清理归档日志的脚本，原理同[2.1](# 2.1 oracle用户打开RMAN工具清理)

脚本内容如下

```shell
#!/bin/bash
echo "----------------------------------------`date`---------------------------------------"
source ~/.bash_profile
rman target / >> /u01/app/del_archlog/delarchive.log <<EOF                                   # 记录日志delarchive.log 可以后期检查是否执行成功
crosscheck archivelog all;                                              # 把无效的expired的archivelog标出来
delete noprompt expired archivelog all;                                 # 直接全部删除过期的归档日志
delete noprompt archivelog all completed before 'sysdate-1';            # 直接删除一天前所有的归档日志
exit
EOF
echo -e "\n"
echo "------------------------------------ FINISHED ------------------------------------"

```

脚本在*/u01/app/del_arch.sh*(注:必须由oracle用户执行，否则无RMAN权限)

```shell
[oracle@rac1 ~]$ cd /u01/app
[oracle@rac1 app]$ ll
total 8
drwxr-xr-x  3 root   oinstall   26 May 27  2020 11.2.0
drwxrwxrwx  2 oracle oinstall   64 Aug 11 11:33 del_archlog
-rwxr-xr-x  1 oracle oinstall  747 Aug 11 11:33 del_arch.sh
drwxrwxr-x 10 grid   oinstall 4096 Jun  9  2020 oracle
drwxrwxr-x  7 oracle oinstall  108 May 27  2020 oracledb
drwxrwx---  6 grid   oinstall  131 May 27  2020 oraInventory
[oracle@rac1 app]$

```

执行方式为

```bash
[oracle@rac1 app]$ sh /u01/app/del_arch.sh
----------------------------------------Fri Aug 11 11:33:27 CST 2023---------------------------------------


------------------------------------ FINISHED ------------------------------------
[oracle@rac1 app]$
```

执行完毕后log日志可在*/u01/app/del_archlog/delarchive.log*查看

```bash
[oracle@rac1 app]$ vim /u01/app/del_archlog/delarchive.log
```

#### 2.3oracle用户定时清理

在rac1上已经建立了定时清理归档日志任务原理为定时执行[2.2](# 2.2 oracle用户使用脚本清理)的脚本,执行时生成的日志位置也一致

定时任务查看

```bash
[oracle@rac1 app]$ crontab -l
* 1 * * * sh /u01/app/del_arc.sh
[oracle@rac1 app]$
```

定时任务修改

```bash
[oracle@rac1 app]$ crontab -e
```

```bash
* 1 * * * sh /u01/app/del_arc.sh
```

定时规则

| 分钟 | 小时 | 日期 | 月份 | 星期                   |
| ---- | ---- | ---- | ---- | ---------------------- |
| 0~59 | 0~23 | 1~31 | 1~12 | 0到7（0或7代表星期日） |

星号（*）：通配符匹配，代表所有可能的值。例如：在小时字段中，一个星号等同于每个小时；在月份字段中，一个星号则等同于每月逗号（,）：在一个字段上指定多个值。例如：“1,2,5,7,8,9”中杠（-）：指定一个值得范围。例如：“2-6”表示“2,3,4,5,6”正斜线（/）：指定时间的间隔频率。例如：“0-23/2”表示每两小时执行一次

\# 每2个小时执行一次脚本

```bash
0 */2 * * * sh /u01/app/del_arc.sh
```

\# 每天凌晨2点执行操作

```bash
0 2 * * * sh /u01/app/del_arc.sh
```

\# 每个工作日的9.AM执行操作

```bash
0 9 * * 1-5 sh /u01/app/del_arc.sh 或 0 9 * * 1,2,3,4,5 sh /u01/app/del_arc.sh
```

#每周六、周日的6:30.pm执行操作

```bash
30 6 * * 0,6 sh /u01/app/del_arc.sh
```

\# 每天22：00.pm-24:00.pm之间每个30min执行操作

```bash
0,30 22-24 * * * sh /u01/app/del_arc.sh
```

linux中提供了8个特殊字符串用来替代crontab命令的前五个字段，这样不但可以节省时间，还可以提高可读性。

| 特殊字符  | 含义                                |
| --------- | ----------------------------------- |
| @reboot   | 在每次启动时运行一次                |
| @yearly   | 每年运行一次，例如：“0 0 1 1 *”   |
| @annually | 与@yearly用法一致                   |
| @monthly  | 每月运行一次，例如：“0 0 1 * *”   |
| @weekly   | 每周运行一次，例如：“0 0 * * 0”   |
| @daily    | 每天运行一次，例如：“0 0 * * *”   |
| @midnight | 与@daily用法一致                    |
| @hourly   | 每小时运行一次，例如：“0 * * * *” |

如每天执行

```bash
@daily sh /u01/app/del_arc.sh
```

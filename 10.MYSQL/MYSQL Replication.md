# MYSQL Replication

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 目录

[TOC]

## 服务器信息

`服务器主机名`|`服务器IP地址`|`功能`
---|---|---
`server2016041901`|`172.31.117.222`|主节点（`Master Node`）
`server86`|`172.31.117.86`|从节点（`Slave Node`）

## `MYSQL` 版本

```bash
# 主从服务器MYSQL版本一致
> mysql -V
mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1
```

## `MYSQL Replication` 操作步骤

> **注意：** 默认 `主-从` 服务器 `mysql` 已经安装完成。若没有安装可以使用 `yum` 等方法安装。

### 主节点操作

> **注意：** 以下操作需要登陆到主节点服务器上执行。

#### 主节点（`Master`）设置权限

> **注意：** 以下操作需要登陆到主节点服务器上执行。

```bash
# 在主节点上为从节点开启操作权限

# 登陆
> mysql -uroot -proot123
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 179985
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# 赋权
mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO root@'%' IDENTIFIED BY 'root123';
Query OK, 0 rows affected (0.43 sec)

# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.04 sec)

mysql> exit;
Bye
```

#### 修改主节点（`Master`）配置

```bash
# 备份配置文件
> cp /etc/my.cnf /etc/my.cnf.raw
> ll /etc/my.cnf*
-rw-r--r-- 1 root root 648 Jul  6 14:11 /etc/my.cnf
-rw-r--r-- 1 root root 564 Jul  6 14:06 /etc/my.cnf.raw

# 在主节点mysql服务器的配置文件（`my.cnf`）中添加如下内容。
# [mysqld]
# ...
# server-id=1
# log-bin=mysql-bin
# ...
> vim /etc/my.cnf
```

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0


key_buffer_size=128M
read_buffer_size=8M
read_rnd_buffer_size=64M
bulk_insert_buffer_size=256M
myisam_sort_buffer_size=256M
myisam_max_sort_file_size=10G
myisam_max_extra_sort_file_size=10G
myisam_repair_threads=1
max_connections=5000
max_allowed_packet=50M
default-storage-engine=INNODB

# mysql-replication config
server-id=1
log-bin=mysql-bin
# mysql-replication -EOF-

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
max_allowed_packet=40M
```

```bash
# 对比修改
> git diff /etc/my.cnf.raw /etc/my.cnf
diff --git a/etc/my.cnf.raw b/etc/my.cnf
index 7512486..afcd480 100644
--- a/etc/my.cnf.raw
+++ b/etc/my.cnf
@@ -18,6 +18,11 @@ max_connections=5000
 max_allowed_packet=50M
 default-storage-engine=INNODB
 
+# mysql-replication config
+server-id=1
+log-bin=mysql-bin
+# mysql-replication -EOF-
+
 [mysqld_safe]
 log-error=/var/log/mysqld.log
 pid-file=/var/run/mysqld/mysqld.pid
```

#### 主节点（`Master`）`MYSQL` 服务重启

> **注意：** 以下操作需要登陆到主节点服务器上执行。

```bash
# 在主节点上重启服务
> service mysqld restart
Stopping mysqld:                                           [  OK  ]
Starting mysqld:                                           [  OK  ]
```

#### 拷贝主节点（`Master`）数据到从节点（`Slave`）中

> **注意：** 以下操作需要登陆到主节点服务器上执行。

```bash
# 进入mysql
> mysql -uroot -proot123
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26
Server version: 5.1.73-log Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# 锁表保证无内容写入
mysql> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.01 sec)

# 查看主节点状态
#!# 需要记住`File`的值，之后配置从节点时会使用
#!# 需要记住`Position`的值，之后配置从节点时会使用
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |   244242 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

mysql> 
```

> **注意：不要退出该`mysql`终端，否则`锁表`操作会取消。请在主节点打开一个新的终端进行如下操作。** 

```bash
# 进入mysql数据目录，打包mysql数据。
# 若不知道数据目录在哪里，可以在配置文件(`/etc/my.cnf`)中查看配置(`datadir`)
> cat /etc/my.cnf | grep datadir
datadir=/var/lib/mysql

# 查看mysql数据目录内容
> ll /var/lib/mysql
total 8720892
drwx------  2 mysql mysql       4096 Jan 12 10:03 activiti
drwx------  2 mysql mysql      16384 May 17 10:58 ceshi
drwx------  2 mysql mysql       4096 Oct 14  2016 goldenSunshine
drwx------  2 mysql mysql       4096 Sep  6  2016 goldenSunShine
-rw-rw----. 1 mysql mysql 8919187456 Jul  6 14:42 ibdata1
-rw-rw----. 1 mysql mysql    5242880 Jul  6 14:42 ib_logfile0
-rw-rw----. 1 mysql mysql    5242880 Jul  6 14:42 ib_logfile1
drwx------. 2 mysql mysql       4096 Apr 19  2016 mysql
-rw-rw----  1 mysql mysql     244242 Jul  6 14:45 mysql-bin.000001
-rw-rw----  1 mysql mysql         19 Jul  6 14:35 mysql-bin.index
srwxrwxrwx  1 mysql mysql          0 Jul  6 14:35 mysql.sock
drwx------  2 mysql mysql      53248 Jul  3 15:31 quality
drwx------  2 mysql mysql      16384 Mar 24 19:48 quality20170324
drwx------  2 mysql mysql      40960 May  9 15:09 quality_222
drwx------  2 mysql mysql      12288 Mar 22 13:46 qualitydd
drwx------  2 mysql mysql      12288 Mar 30 09:24 qualitygf
drwx------  2 mysql mysql      20480 Feb 13 14:32 qualityhx
drwx------  2 mysql mysql      40960 Jul  4 16:57 qualityNew
drwx------  2 mysql mysql       4096 Jun 28 15:03 qualitystatistic
drwx------  2 mysql mysql       4096 Jun 28 15:07 qualityStatistic2
drwx------  2 mysql mysql       4096 Mar  6 14:01 smartVss
drwx------  2 mysql mysql      12288 Feb 10 11:31 smartvsshx
drwx------. 2 mysql mysql       4096 Jan  9 12:32 test

# 执行数据打包命令
#!# 若数据过大可能会需要很久时间
> cd /tmp
> tar -czvf mysql_data.tgz /var/lib/mysql/
[...]

# 检查
> ll /tmp/mysql_data.tgz
> -rw-r--r-- 1 root root 6611979649 Jul  6 16:24 /tmp/mysql_data.tgz

# 拷贝到从节点服务器
> scp -r /tmp/mysql_data.tgz root@172.31.117.86:/var/lib/
[...]

# 之后会登陆到从节点进行配置，注意主节点的那个锁表的终端不要断开
```

### 从节点操作

> **注意：** 以下操作需要登陆到从节点服务器上执行。

#### 完成数据迁移

> **注意：** 以下操作需要登陆到从节点服务器上执行。

```bash
# 停止mysql服务
> service mysqld stop
Stopping mysqld:                                           [  OK  ]

# 备份原有数据目录
> mv /var/lib/mysql /var/lib/mysql.raw
> ll -d /var/lib/mysql*
-rw-r--r--  1 root  root  6611979649 Jul  6 16:32 /var/lib/mysql_data.tgz
drwxr-xr-x. 5 mysql mysql       4096 Oct  8  2016 /var/lib/mysql.raw

# 解压主节点数据
> cd /tmp
> tar -zxvf /var/lib/mysql_data.tgz
[...]

# 检查
> ll /tmp/var/lib/mysql/
total 8720820
drwx------ 2 mysql mysql       4096 Jan 12 10:03 activiti
drwx------ 2 mysql mysql      12288 May 17 10:58 ceshi
drwx------ 2 mysql mysql       4096 Oct 14  2016 goldenSunshine
drwx------ 2 mysql mysql       4096 Sep  6  2016 goldenSunShine
-rw-rw---- 1 mysql mysql 8919187456 Jul  6 14:42 ibdata1
-rw-rw---- 1 mysql mysql    5242880 Jul  6 14:42 ib_logfile0
-rw-rw---- 1 mysql mysql    5242880 Jul  6 14:42 ib_logfile1
drwx------ 2 mysql mysql       4096 Apr 19  2016 mysql
-rw-rw---- 1 mysql mysql     244242 Jul  6 14:45 mysql-bin.000001
-rw-rw---- 1 mysql mysql         19 Jul  6 14:35 mysql-bin.index
drwx------ 2 mysql mysql      49152 Jul  3 15:31 quality
drwx------ 2 mysql mysql      16384 Mar 24 19:48 quality20170324
drwx------ 2 mysql mysql      12288 May  9 15:09 quality_222
drwx------ 2 mysql mysql      12288 Mar 22 13:46 qualitydd
drwx------ 2 mysql mysql      12288 Mar 30 09:24 qualitygf
drwx------ 2 mysql mysql      12288 Feb 13 14:32 qualityhx
drwx------ 2 mysql mysql      20480 Jul  4 16:57 qualityNew
drwx------ 2 mysql mysql       4096 Jun 28 15:03 qualitystatistic
drwx------ 2 mysql mysql       4096 Jun 28 15:07 qualityStatistic2
drwx------ 2 mysql mysql       4096 Mar  6 14:01 smartVss
drwx------ 2 mysql mysql      12288 Feb 10 11:31 smartvsshx
drwx------ 2 mysql mysql       4096 Jan  9 12:32 test

# 拷贝到数据目录位置
> mv /tmp/var/lib/mysql /var/lib/
> ll -d /var/lib/mysql*
drwxr-xr-x  19 mysql mysql       4096 Jul  6 14:35 /var/lib/mysql
-rw-r--r--   1 root  root  6611979649 Jul  6 16:32 /var/lib/mysql_data.tgz
drwxr-xr-x.  5 mysql mysql       4096 Oct  8  2016 /var/lib/mysql.raw


```

#### 修改从节点（`Slave`）配置

> **注意：** 以下操作需要登陆到从节点服务器上执行。

```bash
# 拷贝主节点的配置文件到从节点服务器上
> scp root@172.31.117.222:/etc/my.cnf /etc/my.cnf
[...]

# 备份配置
> cp /etc/my.cnf /etc/my.cnf.raw
> ll /etc/my.cnf*
-rw-r--r--. 1 root root 648 Jul  6 14:15 /etc/my.cnf
-rw-r--r--  1 root root 648 Jul  6 16:39 /etc/my.cnf.raw

# 修改配置
# 在从节点mysql服务器的配置文件（`my.cnf`）中修改和添加如下内容。
# server-id=2 # 设置的值要唯一（和主节点以及其他从节点不一致）
> vim /etc/my.cnf
```

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0


key_buffer_size=128M
read_buffer_size=8M
read_rnd_buffer_size=64M
bulk_insert_buffer_size=256M
myisam_sort_buffer_size=256M
myisam_max_sort_file_size=10G
myisam_max_extra_sort_file_size=10G
myisam_repair_threads=1
max_connections=5000
max_allowed_packet=50M
default-storage-engine=INNODB

# mysql-replication config
server-id=2
# mysql-replication -EOF-

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
max_allowed_packet=40M
```

```bash
# 对比修改后文件
git diff /etc/my.cnf.raw /etc/my.cnf
diff --git a/etc/my.cnf.raw b/etc/my.cnf
index afcd480..d5cc229 100644
--- a/etc/my.cnf.raw
+++ b/etc/my.cnf
@@ -19,8 +19,7 @@ max_allowed_packet=50M
 default-storage-engine=INNODB
 
 # mysql-replication config
-server-id=1
-log-bin=mysql-bin
+server-id=2
 # mysql-replication -EOF-
 
 [mysqld_safe]
```

#### 启动从节点（`Slave`）服务

> **注意：** 以下操作需要登陆到从节点服务器上执行。

```bash
# 启动从节点mysql服务
> service mysqld start
Starting mysqld:                                           [  OK  ]

# 登陆客户端
> mysql -uroot -proot123 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# 配置连接主节点信息
#!# 配置说明：
#!# 1. MASTER_HOST      # 主节点地址
#!# 2. MASTER_USER      # 上文设置权限时指定的用户名
#!# 3. MASTER_PASSWORD  # 上文设置权限时指定的用户密码
#!# 4. MASTER_LOG_FILE  # 上文终端最后执行"SHOW MASTER STATUS;"时结果中`File`的值
#!# 5. MASTER_LOG_POS   # 上文终端最后执行"SHOW MASTER STATUS;"时结果中`Position`的值
mysql> CHANGE MASTER TO MASTER_HOST='172.31.117.222',
    -> MASTER_USER='root',
    -> MASTER_PASSWORD='root123',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=244242;
Query OK, 0 rows affected (0.09 sec)

# 查看设置信息
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 172.31.117.222
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 244242
               Relay_Log_File: mysqld-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 244242
              Relay_Log_Space: 106
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
1 row in set (0.00 sec)

# 启动丛节点服务
> START SLAVE;
Query OK, 0 rows affected (0.00 sec)

# 再次查看丛节点状态
#!# 查看 `Slave_IO_Running` 和 `Slave_SQL_Running` 两个参数均为 `Yes`，说明服务正在运行
> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.31.117.222
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 244242
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 251
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 244242
              Relay_Log_Space: 407
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
1 row in set (0.00 sec)
```

### 主节点解开写入锁

> **注意：该操作是在之前主节点打开并完成加锁命令的那个终端中执行。**

```bash
[...]

#!# 这里应是接着上面主节点未断开的那个终端继续操作

# 执行关闭锁
mysql> UNLOCK TABLES;
Query OK, 0 rows affected (0.09 sec)

# 关闭后退出终端
mysql> exit;
Bye
```

## 检查

### 创建数据库测试

```bash
#!# 登陆主节点上操作
172.31.117.222(主节点)> mysql -uroot -proot123 -e "create database test_mysql_replication;"
172.31.117.222(主节点)> mysql -uroot -proot123 -e "show databases;"
+------------------------+
| Database               |
+------------------------+
           ...
| test_mysql_replication |
+------------------------+

#!# 登陆到从节点上操作
#!# 会发现在从节点上也同样出现刚才创建的数据库
172.31.117.86(从节点)> mysql -uroot -proot123 -e "show databases;"
+------------------------+
| Database               |
+------------------------+
           ...
| test_mysql_replication |
+------------------------+
```

### 创建数据库表测试

```bash
#!# 登陆主节点上操作
172.31.117.222(主节点)> mysql -uroot -proot123 -e "create table test_mysql_replication.first_table(id int);"
172.31.117.222(主节点)> mysql -uroot -proot123 -e "use test_mysql_replication; show tables;"
+----------------------------------+
| Tables_in_test_mysql_replication |
+----------------------------------+
| first_table                      |
+----------------------------------+

#!# 登陆到从节点上操作
#!# 会发现在从节点上也同样出现刚才创建的表
172.31.117.86(从节点)> mysql -uroot -proot123 -e "use test_mysql_replication; show tables;"
+----------------------------------+
| Tables_in_test_mysql_replication |
+----------------------------------+
| first_table                      |
+----------------------------------+
```

### 插入数据测试

```bash
#!# 登陆主节点上操作
172.31.117.222(主节点)> mysql -uroot -proot123 -e "insert into test_mysql_replication.first_table values (001);"
172.31.117.222(主节点)> mysql -uroot -proot123 -e "select * from test_mysql_replication.first_table;"
+------+
| id   |
+------+
|    1 |
+------+

#!# 登陆到从节点上操作
#!# 会发现在从节点上也同样出现刚才创建的表
172.31.117.86(从节点)> mysql -uroot -proot123 -e "select * from test_mysql_replication.first_table;"
+------+
| id   |
+------+
|    1 |
+------+
```

`-EOF-`


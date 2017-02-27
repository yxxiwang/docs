## `MYSQL`安装文档

### 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

### 目录

[TOC]

### 部署环境说明

名称|版本|缩写|默认安装软件组
---|---|---|---
`RedHat-Enterprise-Linux`|`6.5`|`RHEL-6.5`|`Minimal`

#### 需要额外资源信息

资源|作用|说明
---|---|---
操作系统`ISO`文件|部署软件服务器使用|

### 准备

#### 软件服务器

> 部署`软件服务器`，请详见`软件服务器（Repository Server）部署文档`。

### 安装流程

#### 安装 `mysql`

```bash
> yum -y install mysql-server
```

#### 备份 `mysql` 配置文件

```bash
> cp /etc/my.cnf /etc/my.cnf.raw
```

#### 修改 `mysql` 配置文件

> **注意：**
> 根据官方给出的推荐配置进行修改 `或者` 查看下文 `查看修改后配置` 章节。
> 官网配置：http://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_ig_mysql.html#cmig_topic_5_5_2

####  查看修改后配置

```bash
# 查看修改后配置
> cat /etc/my.cnf
```

```bash
[mysqld]
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
# symbolic-links = 0

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system
#and chown the specified folder to the mysql user.
# log_bin=/var/lib/mysql/mysql_binary_log

# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
# binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=STRICT_ALL_TABLES
```

#### 对比 `原始文件` 和 `修改后` 的配置

```bash
# 对比修改后配置 
> git diff /etc/my.cnf.raw /etc/my.cnf

diff --git a/etc/my.cnf.raw b/etc/my.cnf
index fae0fa2..18e59c4 100644
--- a/etc/my.cnf.raw
+++ b/etc/my.cnf
@@ -1,10 +1,45 @@
 [mysqld]
-datadir=/var/lib/mysql
-socket=/var/lib/mysql/mysql.sock
-user=mysql
-# Disabling symbolic-links is recommended to prevent assorted security risks
-symbolic-links=0
+transaction-isolation = READ-COMMITTED
+# Disabling symbolic-links is recommended to prevent assorted security risks;
+# to do so, uncomment this line:
+# symbolic-links = 0
+
+key_buffer = 16M
+key_buffer_size = 32M
+max_allowed_packet = 32M
+thread_stack = 256K
+thread_cache_size = 64
+query_cache_limit = 8M
+query_cache_size = 64M
+query_cache_type = 1
+
+max_connections = 550
+#expire_logs_days = 10
+#max_binlog_size = 100M
+
+#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system
+#and chown the specified folder to the mysql user.
+log_bin=/var/lib/mysql/mysql_binary_log
+
+# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
+binlog_format = mixed
+
+read_buffer_size = 2M
+read_rnd_buffer_size = 16M
+sort_buffer_size = 8M
+join_buffer_size = 8M
+
+# InnoDB settings
+innodb_file_per_table = 1
+innodb_flush_log_at_trx_commit  = 2
+innodb_log_buffer_size = 64M
+innodb_buffer_pool_size = 4G
+innodb_thread_concurrency = 8
+innodb_flush_method = O_DIRECT
+innodb_log_file_size = 512M
 
 [mysqld_safe]
 log-error=/var/log/mysqld.log
 pid-file=/var/run/mysqld/mysqld.pid
+
+sql_mode=STRICT_ALL_TABLES
(END) 
```

#### 设置 `mysql` 自动启动

```bash
> /sbin/chkconfig mysqld on
```

#### 检查 `mysql` 自动启动

> **注意：** `2-5` 项均为 `on`

```bash
> /sbin/chkconfig --list mysqld
mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

#### 启动 `mysql`

```bash
> service mysqld start
Initializing MySQL database:  Installing MySQL system tables...
OK
Filling help tables...
OK

[...]

Starting mysqld:                                           [  OK  ]
```

#### 设置 `mysql` 初始密码

> **问题&答案**
> - 1. Enter current password for root (enter for none): `回车`
> - 2. Set root password? [Y/n] `Y`
> - 3. New password: `root123`
> - 4. Re-enter new password: `root123`
> - 5. Remove anonymous users? [Y/n] `Y`
> - 6. Disallow root login remotely? [Y/n] `n`
> - 7. Remove test database and access to it? [Y/n] `Y`
> - 8. Reload privilege tables now? [Y/n] `Y`
> 
> **注意：** 这里输入的密码需要记住，之后会用到。

```bash
> /usr/bin/mysql_secure_installation

[...]

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

[...]

Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!

[...]

Remove anonymous users? [Y/n] Y
 ... Success!

[...]

Disallow root login remotely? [Y/n] n
 ... skipping.

[...]

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

[...]

Reload privilege tables now? [Y/n] Y
 ... Success!

[...]

All done!  
```

#### 设置 `mysql` 访问权限

```bash
# 登陆`mysql`
> mysql -uroot -proot123
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.1.71-log Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# 赋权
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root123' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 退出客户端
mysql> exit;
Bye
```

### 验证

#### 本地登陆

```bash
> mysql -uroot -proot123 -hlocalhost -e 'show databases;'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
+--------------------+
```

#### 其他服务器登陆

> 登陆到其他服务器上执行。

```bash
> mysql -uroot -proot123 -h10.0.3.45 -e 'show databases;'
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
+--------------------+
```

`-EOF-`



# 使用已编译文件安装 MYSQL 5.7

> 参考文档: https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 目录

[TOC]

## 安装步骤

```bash
> groupadd mysql
> useradd -r -g mysql -s /bin/false mysql
> cd /usr/local
> ln -s /ccicall/opt/mysql-5.7.16-linux-glibc2.5-x86_64/ mysql
> cd mysql
> mkdir mysql-files
> chmod 750 mysql-files
> chown -R mysql .
> chgrp -R mysql .
> yum -y install libaio libaio-devel
```

## 修改配置

```bash
# 修改配置（改成和下面一致）
> vim /etc/my.cnf
```

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
port=3306
socket=/var/lib/mysql/mysql.sock
```

## 设置环境变量

```bash
# 在`/etc/profile`文件中添如下配置
# export MYSQL_HOME=/ccicall/opt/mysql-5.7.16-linux-glibc2.5-x86_64
# export PATH=${MYSQL_HOME}/bin:${PATH}
> vim /etc/profile
[...]
export MYSQL_HOME=/ccicall/opt/mysql-5.7.16-linux-glibc2.5-x86_64
export PATH=${MYSQL_HOME}/bin:${PATH}

# 刷新配置
> source /etc/profile

# 检查
> mysql --version
mysql  Ver 14.14 Distrib 5.7.16, for linux-glibc2.5 (x86_64) using  EditLine wrapper
```

## 初始化

```bash
> bin/mysqld --initialize --user=mysql
> mkdir -p /var/run/mysqld
> chown -R mysql:mysql /var/run/mysqld
> bin/mysqld_safe --user=mysql &

# 拷贝以后就可以使用`service`命令启停服务了，例如重启服务`service mysql.server restart`
> cp support-files/mysql.server /etc/init.d/mysql.server
```

## 创建用户以及赋权

```bash
# 创建用户
> mysql -uroot -proot123 -e "CREATE USER 'ccicall'@'%' IDENTIFIED BY 'work123';"

# 授权
> mysql -uroot -proot123 -e "GRANT ALL ON *.* TO 'ccicall'@'%';"

# 检查
> mysql -uccicall -pwork123 mysql -e "show tables;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
```

## 问题解决

### `Cant connect to local MySQL server through socket '/tmp/mysql.sock'`

```bash
> mysql -uroot
ERROR 2002 (HY000): Cant connect to local MySQL server through socket '/tmp/mysql.sock' (2)

# 在`/etc/my.cnf`文件中，添加`[client]`配置并指定`socket`和`[mysqld]`配置的`socket`配置一致
> vim /etc/my.cnf
```

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
port=3306
socket=/var/lib/mysqld/mysqld.sock
```

### `root`用户密码忘记

> 启动 `mysqld_safe --skip-grant-tables &` 需要保证当前没有 `mysql server` 启动

```bash
# 首先需要停止`mysqld`服务器
# 启动无需密码登陆模式
> mysqld_safe --skip-grant-tables &
# 进入数据库
> mysql -uroot
# 修改`root`用户密码
mysql> update mysql.user set authentication_string=PASSWORD('root123') where User='root';
mysql> flush privileges;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
mysql> exit;
```

> **特别注意：记住修改后要停止`--skip-grant-tables`的`mysqld`服务**

`-EOF-`

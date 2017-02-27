## `REDIS`安装文档

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

#### 安装必要组件

> **注意：**必要组件一般使用`yum`安装，所以这里依赖`软件服务器（Repository Server）`，若没有搭建请详见`软件服务器（Repository Server）部署文档`。

##### 组件列表

名称|作用
---|---
`openssh-clients`|`传输工具（scp）`
`vim`|`编辑工具`
`git`|`版本管理工具`
`gcc`|`编译工具`

##### 安装方法

###### 一般组件安装（使用`yum`）

```bash
# 切换成为`root`用户
> su - 
Password: 

> yum -y install openssh-clients vim git gcc
[...]
```

### 安装流程

#### 获取组件

```bash
# 切换账户
> su -

> cd /opt/

> scp root@10.0.3.45:/Archive/Project_Dadi/02.sources/redis-2.8.13.tar.gz ./  
> ll redis-2.8.13.tar.gz 
-rw-r--r--. 1 root root 1227538 Feb  9 16:51 redis-2.8.13.tar.gz

# 解压
> tar -zxvf redis-2.8.13.tar.gz
> ll redis-2.8.13
total 124
-rw-rw-r--. 1 root root 22424 Jul 14  2014 00-RELEASENOTES
-rw-rw-r--. 1 root root    52 Jul 14  2014 BUGS
-rw-rw-r--. 1 root root  1439 Jul 14  2014 CONTRIBUTING
-rw-rw-r--. 1 root root  1487 Jul 14  2014 COPYING
drwxrwxr-x. 6 root root  4096 Jul 14  2014 deps
-rw-rw-r--. 1 root root    11 Jul 14  2014 INSTALL
-rw-rw-r--. 1 root root   151 Jul 14  2014 Makefile
-rw-rw-r--. 1 root root  4223 Jul 14  2014 MANIFESTO
-rw-rw-r--. 1 root root  4404 Jul 14  2014 README
-rw-rw-r--. 1 root root 32213 Jul 14  2014 redis.conf
-rwxrwxr-x. 1 root root   271 Jul 14  2014 runtest
-rwxrwxr-x. 1 root root   281 Jul 14  2014 runtest-sentinel
-rw-rw-r--. 1 root root  6226 Jul 14  2014 sentinel.conf
drwxrwxr-x. 2 root root  4096 Jul 14  2014 src
drwxrwxr-x. 9 root root  4096 Jul 14  2014 tests
drwxrwxr-x. 3 root root  4096 Jul 14  2014 utils
```

#### 编译安装

```bash
# 进入目录
> cd /opt/redis-2.8.13

# 编译依赖组件
> make MALLOC=libc
make[1]: Entering directory `/opt/redis-2.8.13/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-dump redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html
(cd ../deps && make distclean)

[...]

Hint: To run 'make test' is a good idea ;)

make[1]: Leaving directory `/opt/redis-2.8.13/src'

# 编译安装redis
> make PREFIX=/usr/local/redis install
cd src && make install
make[1]: Entering directory `/opt/redis-2.8.13/src'

Hint: To run 'make test' is a good idea ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/opt/redis-2.8.13/src'

# 验证
> ll /usr/local/redis/bin/
total 2432
-rwxr-xr-x. 1 root root  231166 Feb  9 16:57 redis-benchmark
-rwxr-xr-x. 1 root root   22161 Feb  9 16:57 redis-check-aof
-rwxr-xr-x. 1 root root   45363 Feb  9 16:57 redis-check-dump
-rwxr-xr-x. 1 root root  327369 Feb  9 16:57 redis-cli
-rwxr-xr-x. 1 root root 1854028 Feb  9 16:57 redis-server
```

#### 配置

```bash
# 创建目录
> mkdir -p /etc/redis /var/redis /var/redis/log /var/redis/run /var/redis/6379
 
# 拷贝配置文件
> cp redis.conf /etc/redis/6379.conf
> ll /etc/redis/6379.conf
-rw-r--r--. 1 root root 32213 Feb  9 17:01 /etc/redis/6379.conf

# - 修改第37行，`daemonize` 将 `no` 修改为 `yes`
# - 修改第41行，`pidfile` 将 `/var/run/redis.pid` 修改为 `/var/redis/run/redis_6379.pid`
# - 修改第103行，`logfile` 将 `""` 修改为 `/var/redis/log/redis_6379.log`
# - 修改第187行，`dir` 将 `./` 修改为 `/var/redis/6379`
> vim /etc/redis/6379.conf

# 对比修改后文件
> git diff /opt/redis-2.8.13/redis.conf /etc/redis/6379.conf 
diff --git a/opt/redis-2.8.13/redis.conf b/etc/redis/6379.conf
index 6efb6ac..c176ce7 100644
--- a/opt/redis-2.8.13/redis.conf
+++ b/etc/redis/6379.conf
@@ -34,11 +34,11 @@
 
 # By default Redis does not run as a daemon. Use 'yes' if you need it.
 # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
-daemonize no
+daemonize yes
 
 # When running daemonized, Redis writes a pid file in /var/run/redis.pid by
 # default. You can specify a custom pid file location here.
-pidfile /var/run/redis.pid
+pidfile /var/redis/run/redis_6379.pid
 
 # Accept connections on the specified port, default is 6379.
 # If port 0 is specified Redis will not listen on a TCP socket.
@@ -100,7 +100,7 @@ loglevel notice
 # Specify the log file name. Also the empty string can be used to force
 # Redis to log on the standard output. Note that if you use standard
 # output for logging but daemonize, logs will be sent to /dev/null
-logfile ""
+logfile /var/redis/log/redis_6379.log
 
 # To enable logging to the system logger, just set 'syslog-enabled' to yes,
 # and optionally update the other syslog parameters to suit your needs.
@@ -184,7 +184,7 @@ dbfilename dump.rdb
 # The Append Only File will also be created inside this directory.
 # 
 # Note that you must specify a directory here, not a file name.
-dir ./
+dir /var/redis/6379
 
 ################################# REPLICATION #################################
```

#### 写入环境变量

```bash
# 备份
> cp /etc/profile /etc/profile.raw
> ll /etc/profile*
-rw-r--r--. 1 root root 1796 Aug 20  2013 /etc/profile
-rw-r--r--. 1 root root 1796 Feb  9 17:11 /etc/profile.raw

[...]

# 添加`redis`（在文件最后添加）
> vim /etc/profile
```

```bash
export REDIS_HOME=/usr/local/redis
export PATH=${REDIS_HOME}/bin:${PATH}
```

```bash
# 对比修改后文件
> git diff /etc/profile.raw /etc/profile
diff --git a/etc/profile.raw b/etc/profile
index 1b82ae6..80350ca 100644
--- a/etc/profile.raw
+++ b/etc/profile
@@ -76,3 +76,7 @@ done
 
 unset i
 unset -f pathmunge
+
+export REDIS_HOME=/usr/local/redis
+export PATH=${REDIS_HOME}/bin:${PATH}
+ 

# 使配置生效
> source /etc/profile
```

#### 启动

```bash
> su - 
> redis-server /etc/redis/6379.conf
```

#### 验证

```bash
> su - ccicall
> redis-cli -v
redis-cli 2.8.13

# 删除组件包
> su - 
> rm -rf /opt/redis-2.8.13.tar.gz 
> ll /opt/
total 4
drwxrwxr-x. 6 root root 4096 Feb  9 17:13 redis-2.8.13
```

`-EOF-`


## `MongoDB`安装文档

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

##### 安装方法

###### 一般组件安装（使用`yum`）

```bash
# 切换成为`root`用户
> su - 
Password: 

> yum -y install openssh-clients vim git
[...]
```

### 安装流程

> **！！特别注意：在安装之前要保证所有使用的服务器`防火墙`和`SELINUX`已经全部关闭！否则会有问题！**

```bash
# 切换用户
> su -

# 安装`open-ssl` 
> yum -y install openssl
[...]

# 切换目录
> cd /opt/
> pwd
/opt

# 获取软件
> scp root@10.0.3.45:/Archive/Project_Dadi/02.sources/mongodb/mongodb-linux-x86_64-rhel62-3.2.5.tgz ./
> ll mongodb-linux-x86_64-rhel62-3.2.5.tgz 
-rw-r--r-- 1 root root 68593830 Feb 16 13:50 mongodb-linux-x86_64-rhel62-3.2.5.tgz

# 解压
> tar -zxvf mongodb-linux-x86_64-rhel62-3.2.5.tgz
[...]

# 检查
> ll mongodb-linux-x86_64-rhel62-3.2.5
total 100
drwxr-xr-x 2 root root  4096 Feb 16 13:50 bin
-rw-r--r-- 1 root root 34520 Apr 13  2016 GNU-AGPL-3.0
-rw-r--r-- 1 root root 16726 Apr 13  2016 MPL-2
-rw-r--r-- 1 root root  1359 Apr 13  2016 README
-rw-r--r-- 1 root root 35910 Apr 13  2016 THIRD-PARTY-NOTICES

# 拷贝到`/usr/local/`目录下
> cp -r mongodb-linux-x86_64-rhel62-3.2.5 /usr/local/mongodb
> ll /usr/local/mongodb/
total 100
drwxr-xr-x 2 root root  4096 Feb 16 13:59 bin
-rw-r--r-- 1 root root 34520 Feb 16 13:59 GNU-AGPL-3.0
-rw-r--r-- 1 root root 16726 Feb 16 13:59 MPL-2
-rw-r--r-- 1 root root  1359 Feb 16 13:59 README
-rw-r--r-- 1 root root 35910 Feb 16 13:59 THIRD-PARTY-NOTICES

# 创建数据和日志目录
> mkdir -p /usr/local/mongodb/data/db/ /usr/local/mongodb/logs/
> ll /usr/local/mongodb/data/db/ /usr/local/mongodb/logs/
/usr/local/mongodb/data/db/:
total 0

/usr/local/mongodb/logs/:
total 0

# 设置开机自动启动
#!# 注意该命令每台服务器只能执行一次
> echo "/usr/local/mongodb/bin/mongod --dbpath=/usr/local/mongodb/data/db --logpath=/usr/local/mongodb/logs/mongodb.log --fork" >> /etc/rc.local

# 检查
> cat /etc/rc.local | grep mongodb
/usr/local/mongodb/bin/mongod --dbpath=/usr/local/mongodb/data/db --logpath=/usr/local/mongodb/logs/mongodb.log --fork

# 启动
> /usr/local/mongodb/bin/mongod --dbpath=/usr/local/mongodb/data/db --logpath=/usr/local/mongodb/logs/mongodb.log --fork
about to fork child process, waiting until server is ready for connections.
forked process: 12466
child process started successfully, parent exiting

# 进入`mongodb`
> /usr/local/mongodb/bin/mongo
MongoDB shell version: 3.2.5
connecting to: test
[...]

# 创建`smartv`
> use smartv
switched to db smartv

# 退出
> exit
bye

# 设置环境变量（在文件最后添加）
> vim /etc/profile
```

```bash
export MONGODB_HOME=/usr/local/mongodb
export PATH=${MONGODB_HOME}/bin:${PATH}
```

```bash
# 使配置生效
> source /etc/profile

# 验证
> mongo --version 
MongoDB shell version: 3.2.5

# 删除软件包
> rm -rf /opt/mongodb-linux-x86_64-rhel62-3.2.5.tgz 
> ll /opt/
[...]
drwxr-xr-x   3 root         root                4096 Apr 18 09:21 mongodb-linux-x86_64-rhel62-3.2.5
[...]
```

`-EOF-`


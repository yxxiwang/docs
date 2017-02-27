## `FTP`安装文档

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

# 执行安装命令
> yum -y install vsftpd
Loaded plugins: product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
MooseFS                                                                             |  951 B     00:00     
rhel-media                                                                          | 3.9 kB     00:00     
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package vsftpd.x86_64 0:2.2.2-11.el6_4.1 will be installed
--> Finished Dependency Resolution

[...]

Installed:
  vsftpd.x86_64 0:2.2.2-11.el6_4.1                                                                         

Complete!

# 检查
> vsftpd -v
vsftpd: version 2.2.2

# 创建`ftp`数据目录
> mkdir -p /data/ftpdir
> ll -d /data/ftpdir/
drwxr-xr-x 2 root root 4096 Feb 16 09:50 /data/ftpdir/

# 赋权
> chmod -R 777 /data/ftpdir/
> ll -d /data/ftpdir/
drwxrwxrwx 2 root root 4096 Feb 16 09:50 /data/ftpdir/

# 备份`ftp`配置
> cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.raw
> ll /etc/vsftpd/vsftpd.conf*
-rw------- 1 root root 4599 Feb 13  2013 /etc/vsftpd/vsftpd.conf
-rw------- 1 root root 4599 Feb 16 09:44 /etc/vsftpd/vsftpd.conf.raw

# 修改`ftp`配置
# 修改第12行，`anonymous_enable`，将`YES`改为`NO`
# 在第14行添加，`local_root`，为`local_root=/data/ftpdir`
vim /etc/vsftpd/vsftpd.conf

# 对比修改后配置文件
> git diff /etc/vsftpd/vsftpd.conf.raw /etc/vsftpd/vsftpd.conf
diff --git a/etc/vsftpd/vsftpd.conf.raw b/etc/vsftpd/vsftpd.conf
index 6cec64b..45676a5 100644
--- a/etc/vsftpd/vsftpd.conf.raw
+++ b/etc/vsftpd/vsftpd.conf
@@ -9,7 +9,9 @@
 # capabilities.
 #
 # Allow anonymous FTP? (Beware - allowed by default if you comment this out).
-anonymous_enable=YES
+anonymous_enable=NO
+
+local_root=/data/ftpdir
 #
 # Uncomment this to allow local users to log in.
 local_enable=YES
 
# 创建`ftp`使用用户
#!# 这里提示信息的意思是`home`目录已经存在，可以忽略。
> useradd -d /data/ftpdir/ -s /sbin/nologin ftpuser
useradd: warning: the home directory already exists.
Not copying any file from skel directory into it.

# 检查
#!# 代码如果不是`501`没有问题
> cat /etc/passwd | grep ftpuser
ftpuser:x:501:501::/data/ftpdir/:/sbin/nologin

# 为`ftp`用户设置密码: ftpuser
> passwd ftpuser
Changing password for user ftpuser.
New password:  # 这里输入 ftpuser
BAD PASSWORD: it is based on a dictionary word
BAD PASSWORD: is too simple
Retype new password:  # 再次输入 ftpuser
passwd: all authentication tokens updated successfully.

# 为`/data/ftpdir`更换`用户`和`用户组`
> chown -R ftpuser:ftpuser /data/ftpdir/
> ll -d /data/ftpdir/
drwxrwxrwx 2 ftpuser ftpuser 4096 Feb 16 09:50 /data/ftpdir/

# 启动服务
> service vsftpd start
Starting vsftpd for vsftpd:                                [  OK  ]
``` 

> 验证，进入页面：ftp://10.0.3.48/
> 输入用户名/密码：ftpuser/ftpuser

![](media/14871509385277/14872106604230.jpg)

`-EOF-`



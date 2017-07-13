# 基于RHEL-6.5的ClouderaEnterprise-5.6.0环境搭建完全手册

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 目录

[TOC]

## 部署环境说明

### 版本信息

名称|版本|缩写
---|---|---
`RedHat-Enterprise-Linux`|`6.5`|`RHEL-6.5`
`Cloudera-Enterprise`|`5.6.0`|
`Cloudera-Manager`|`5.6.0`|`CM-5.6.0`
`Cloudera-Distribution-including-apache-Hadoop`|`5.6.0`|`CDH-5.6.0`

### 服务器信息

角色|`IP`地址|主机名(`HOSTNAME`)|操作系统|配置
---|---|---|---|---
集群服务器|`10.0.1.49`|`server349`|`RHEL-6.5`|`10 cores, 64GB memory`
集群服务器|`10.0.1.51`|`server351`|`RHEL-6.5`|`10 cores, 64GB memory`
集群服务器|`10.0.1.52`|`server352`|`RHEL-6.5`|`10 cores, 64GB memory`
集群服务器|`10.0.1.53`|`server353`|`RHEL-6.5`|`10 cores, 64GB memory`
集群服务器|`10.0.1.54`|`server354`|`RHEL-6.5`|`10 cores, 64GB memory`
软件服务器|`10.0.1.49`|`server349`|`RHEL-6.5`|`-`

### 参考资料

- 官方安装方法B：http://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_ig_install_path_b.html

### 资源下载

- CDH相关、通过parcels方式安装：http://archive.cloudera.com/cdh5/parcels/5.6.0/
- Cloudera Manager相关，包含server、agent、oracle-jdk等：https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.6.0/ 

<!-- more -->

### 依赖软件

> **注意：**由于若安装操作系统时选择的是`Minimal`（最小化）安装。那么在执行下文中某些命令时会出现该命令不存在的错误（比如`wget`, `vim`, `scp`等）。所以这里给出一个基于`Minimal`的需要安装的软件列表，若在下文中碰到该命令不存在时，可以在已经完成`搭建软件服务器`的情况下，通过`yum`安装。例如需要安装`wget`则执行`yum -y install wget`。

#### 组件列表

名称|作用
---|---
`openssh-clients`|`传输工具（scp）`
`vim`|`编辑工具`
`unzip`|`解压工具`
`mysql`|`数据库工具`
`git`|`版本管理工具`
`wget`|`下载工具`

## 准备工作

### 搭建软件服务器（ `Repository Server` ）

> **注意：** 搭建软件服务器前，系统 `必须` 有 `httpd` 软件。

> **注意：** 该章节适用于没有 `Archvie.zip` 和 `repo` 服务器的情况。
> - 如果已经存在 `Archive.zip`，可直接解压 `Archive.zip` 文件到 `/` 目录后，并执行   `RedHat Enterprise Linxu Media Source` 章节即可。
> - 如果已经存在 `repo` 服务器，则直接将 `repo` 文件拷贝到 `/etc/yum.repo.d/` 目录即可。

#### 创建 `repo` 目录

- 1. 登陆 `repo` 服务器

> **注意：** 这里忽略登陆服务器的描述。

```bash
# 登陆后，查看 `repo` 服务器 `ip` 地址
> /sbin/ifconfig 
[...]

eth1      Link encap:Ethernet  HWaddr A0:36:9F:8C:2B:CF  
          inet addr:10.0.3.49  Bcast:10.0.3.255  Mask:255.255.255.0
          inet6 addr: fe80::a236:9fff:fe8c:2bcf/64 Scope:Link
          
[...]

# 登陆后，查看 `repo` 服务器操作用户
> whoami
root
```

- 2. 创建 `repo` 目录

```bash
# 创建 `repo` 目录
> mkdir -p /Archive

# 检查已经创建的 `repo` 目录
> ll -d /Archive/
drwxr-xr-x. 6 root root 4096 Nov 23 18:25 /Archive/
```

#### 获取软件资源

名称|说明
---|---
`RedHat Enterprise Linxu Media Source`|系统光盘软件：`mysql-server`,`python`,`...`
`Cloudera Enterprise - CDH - Parcels`|`CDH`相关软件：`hadoop`,`hbase`,`hive`,`...`
`Cloudera Enterprise - CM`|`CM`相关软件：`agent`,`server`,`oracle-jdk`,`...`
`Scripts`|执行脚本：`创建信任关系`,`批量执行命令`,`...`
`Resources`|其他资源：`mysql-jdbc-driver`,`apache配置文件`,`...`

##### `RedHat Enterprise Linxu Media Source`

- 1. 获得系统光盘（`ISO`）

> **注意：** 可以有多种方式获取系统光盘的 `ISO` 文件，所以如何获得 `ISO` 文件这里不做具体描述。
> - 可以将系统光盘插入服务器光驱并在服务器文件系统中挂载光驱后，到挂载目录中拷贝 `ISO` 文件。（ 参考教程：http://jingyan.baidu.com/article/e52e3615a9c19440c60c5121.html ）
> - 可以通过官方网站下载对应版本的操作系统 `ISO` 文件。
> 
> 推荐使用通过系统光盘获取 `ISO` 文件方式。
> 
> **默认已经将 `ISO` 文件，下载到 `/root` 目录中。**

```bash
# 查看已经获取 `ISO` 文件路径
> ll /root/rhel-server-6.5-x86_64-dvd.iso 
-rwxr-xr-x 1 root root 3853516800 Nov 25 09:38 /root/rhel-server-6.5-x86_64-dvd.iso
```

- 2. 创建目录

> **注意：** 需要分别创建 `存放` 和 `挂载` `ISO` 文件的目录。
 
- 2.1. 创建存放 `ISO` 文件目录

```bash
# 执行创建目录命令
> mkdir -p /Archive/RedHat-Enterprise-Linux/DVD

# 查看已经创建的目录
> ll -d /Archive/RedHat-Enterprise-Linux/DVD/
drwxr-xr-x 2 root root 4096 Nov 23 16:02 /Archive/RedHat-Enterprise-Linux/DVD/
```

- 2.2. 创建挂载 `ISO` 文件目录

```bash
# 执行创建目录命令
> mkdir -p /Archive/RedHat-Enterprise-Linux/6.5

# 查看已经创建目录
> ll -d /Archive/RedHat-Enterprise-Linux/6.5/
dr-xr-xr-x 12 root root 8192 Nov 12  2013 /Archive/RedHat-Enterprise-Linux/6.5/
```

- 3. 拷贝 `ISO` 文件到创建目录中

```bash
# 执行拷贝命令
> cp /root/rhel-server-6.5-x86_64-dvd.iso /Archive/RedHat-Enterprise-Linux/DVD/

# 查看已经拷贝完成的文件
> ll /Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso
-rwxr-xr-x. 1 root root 3853516800 Nov 25 09:44 /Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso

# 删除原始文件
> rm -rf /root/rhel-server-6.5-x86_64-dvd.iso
```

- 4. 设置自动挂载配置

> **注意：** 将挂载信息写入 `/etc/fstab` 文件中，以便可以实现服务器重启后自动挂载。

- 4.1. 备份挂载配置文件 `/etc/fstab` 

```bash
> cp /etc/fstab /etc/fstab.raw
```

- 4.2. 修改挂载配置文件 `/etc/fstab`

> **注意：** 该命令只能执行一次。若重复执行，那么每次执行都会在 `/etc/fstab` 文件的最后一行写入一行配置。

```bash
# 将挂载配置写入 `/etc/fstab` 文件中
> echo "/Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso /Archive/RedHat-Enterprise-Linux/6.5 iso9660 defaults,ro,loop 0 0" >> /etc/fstab
```

- 4.3. 查看写入后 `/etc/fstab` 文件内容

```bash
> cat /etc/fstab
```

```bash
#
# /etc/fstab
# Created by anaconda on Fri Nov 18 17:51:46 2016
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=2d924c15-6347-42ed-bbb1-f1db299c7d51 /                       ext4    defaults        1 1
UUID=b549c849-d9c7-4d8d-b6ab-0765787db3dc /boot                   ext4    defaults        1 2
UUID=8504-35B7          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
UUID=4d4cecda-db84-4306-964d-87891053a665 /home                   ext4    defaults        1 2
UUID=f625eb8e-91c4-4181-be53-c9e3f07304b7 swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso /Archive/RedHat-Enterprise-Linux/6.5 iso9660 defaults,ro,loop 0 0
```

- 4.4. 和原始文件对比

> **注意：** 执行该命令需要安装 `git` 。

```bash
> git diff /etc/fstab.raw /etc/fstab
diff --git a/etc/fstab.raw b/etc/fstab
index ab8f55f..2e95b15 100644
--- a/etc/fstab.raw
+++ b/etc/fstab
@@ -15,3 +15,4 @@ tmpfs                   /dev/shm                tmpfs   defaults        0 0
 devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
 sysfs                   /sys                    sysfs   defaults        0 0
 proc                    /proc                   proc    defaults        0 0
+/Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso /Archive/RedHat-Enterprise-Linux/6.5 iso9660 defaults,ro,loop 0 0
```

- 5. 挂载 `ISO` 文件

```bash
# 执行挂载命令
> mount -a

# 查看挂载信息
> df -hk
[...]
/Archive/RedHat-Enterprise-Linux/DVD/rhel-server-6.5-x86_64-dvd.iso     3762278  3762278           0 100% /Archive/RedHat-Enterprise-Linux/6.5

# 查看挂载目录内容
> ll /Archive/RedHat-Enterprise-Linux/6.5/
total 2348
dr-xr-xr-x 3 root root   2048 Nov 12  2013 EFI
lr-xr-xr-x 1 root root      7 Nov 12  2013 EULA -> EULA_en
-r--r--r-- 3 root root  10726 Nov  7  2012 EULA_de
-r--r--r-- 3 root root   8724 Nov  7  2012 EULA_en
-r--r--r-- 3 root root  10846 Nov  7  2012 EULA_es
-r--r--r-- 3 root root  10682 Nov  7  2012 EULA_fr
-r--r--r-- 3 root root  10497 Nov  7  2012 EULA_it
-r--r--r-- 3 root root  13173 Nov  7  2012 EULA_ja
-r--r--r-- 3 root root   9841 Nov  7  2012 EULA_ko
-r--r--r-- 3 root root  10033 Nov  7  2012 EULA_pt
-r--r--r-- 3 root root   7306 Nov  7  2012 EULA_zh
-r--r--r-- 3 root root  18092 Jun 30  2010 GPL
dr-xr-xr-x 3 root root   2048 Nov 12  2013 HighAvailability
dr-xr-xr-x 3 root root   2048 Nov 12  2013 images
dr-xr-xr-x 2 root root   2048 Nov 12  2013 isolinux
dr-xr-xr-x 3 root root   2048 Nov 12  2013 LoadBalancer
-r--r--r-- 2 root root    114 Nov 12  2013 media.repo
dr-xr-xr-x 2 root root 679936 Nov 12  2013 Packages
-r--r--r-- 2 root root  16435 Sep  2  2010 README
-r--r--r-- 3 root root  79246 Oct 25  2013 RELEASE-NOTES-as-IN.html
-r--r--r-- 3 root root  81189 Oct 25  2013 RELEASE-NOTES-bn-IN.html
-r--r--r-- 3 root root  49152 Oct 29  2013 RELEASE-NOTES-de-DE.html
-r--r--r-- 3 root root  45660 Oct 29  2013 RELEASE-NOTES-en-US.html
-r--r--r-- 3 root root  49114 Oct 25  2013 RELEASE-NOTES-es-ES.html
-r--r--r-- 3 root root  52407 Oct 29  2013 RELEASE-NOTES-fr-FR.html
-r--r--r-- 3 root root  74554 Oct 29  2013 RELEASE-NOTES-gu-IN.html
-r--r--r-- 3 root root  76054 Oct 25  2013 RELEASE-NOTES-hi-IN.html
-r--r--r-- 3 root root  48477 Oct 25  2013 RELEASE-NOTES-it-IT.html
-r--r--r-- 3 root root  53592 Oct 25  2013 RELEASE-NOTES-ja-JP.html
-r--r--r-- 3 root root  87835 Oct 25  2013 RELEASE-NOTES-kn-IN.html
-r--r--r-- 3 root root  48292 Oct 25  2013 RELEASE-NOTES-ko-KR.html
-r--r--r-- 3 root root 101040 Oct 29  2013 RELEASE-NOTES-ml-IN.html
-r--r--r-- 3 root root  77763 Oct 25  2013 RELEASE-NOTES-mr-IN.html
-r--r--r-- 3 root root  82766 Oct 29  2013 RELEASE-NOTES-or-IN.html
-r--r--r-- 3 root root  74777 Oct 25  2013 RELEASE-NOTES-pa-IN.html
-r--r--r-- 3 root root  48515 Oct 29  2013 RELEASE-NOTES-pt-BR.html
-r--r--r-- 3 root root  54156 Oct 29  2013 RELEASE-NOTES-ru-RU.html
-r--r--r-- 3 root root   5125 May  5  2010 RELEASE-NOTES-si-LK.html
-r--r--r-- 3 root root  94236 Oct 25  2013 RELEASE-NOTES-ta-IN.html
-r--r--r-- 3 root root  83646 Oct 25  2013 RELEASE-NOTES-te-IN.html
-r--r--r-- 3 root root  88640 Oct 25  2013 RELEASE-NOTES-zh-CN.html
-r--r--r-- 3 root root  92745 Oct 30  2013 RELEASE-NOTES-zh-TW.html
dr-xr-xr-x 2 root root   4096 Nov 12  2013 repodata
dr-xr-xr-x 3 root root   2048 Nov 12  2013 ResilientStorage
-r--r--r-- 3 root root   3375 Oct 30  2013 RPM-GPG-KEY-redhat-beta
-r--r--r-- 3 root root   3211 Oct 30  2013 RPM-GPG-KEY-redhat-release
dr-xr-xr-x 3 root root   2048 Nov 12  2013 ScalableFileSystem
dr-xr-xr-x 3 root root   2048 Nov 12  2013 Server
-r--r--r-- 1 root root  11414 Nov 12  2013 TRANS.TBL
```

- 6. 创建 `rhel 6.5 repo` 文件

```bash
# 进入目录
> cd /Archive/RedHat-Enterprise-Linux/

# 创建 `rhel-6.5-media.repo` 文件，内容如下:
> cat rhel-6.5-media.repo 
```

```bash
[rhel-media]
name=Red Hat Enterprise Linux 6.5
baseurl=http://archive.centrin.com.cn/RedHat-Enterprise-Linux/6.5/
enabled=1
gpgcheck=1
gpgkey=http://archive.centrin.com.cn/RedHat-Enterprise-Linux/6.5/RPM-GPG-KEY-redhat-release
```

- 7. 查看 `RedHat-Enterprise-Linux` 目录全部内容

```bash
> ll /Archive/RedHat-Enterprise-Linux/
total 16
dr-xr-xr-x 12 root root 8192 Nov 12  2013 6.5
drwxr-xr-x  2 root root 4096 Nov 23 16:02 DVD
-rwxr-xr-x  1 root root  227 Nov 23 16:14 rhel-6.5-media.repo
```

##### `Cloudera Enterprise - CDH - Parcels` 

- 1. 创建目录

```bash
# 创建目录
> mkdir -p /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0

# 查看创建目录
> ll -d /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0
drwxr-xr-x. 2 root root 4096 Nov 23 14:23 /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0

# 进入目录
> cd /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0/
```

- 2. 下载 `CDH - Parcels` 文件

> **注意：** 要下载对应版本的内容。

```bash
# 进入目录
> cd /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0/

# 下载 `parcel` 文件
> wget http://archive.cloudera.com/cdh5/parcels/5.6.0/CDH-5.6.0-1.cdh5.6.0.p0.45-el6.parcel ./
[...]

# 下载 `parcel - sha1` 校验文件
> wget http://archive.cloudera.com/cdh5/parcels/5.6.0/CDH-5.6.0-1.cdh5.6.0.p0.45-el6.parcel.sha1 ./
[...]

# 下载 `manifest.json` 文件
> wget http://archive.cloudera.com/cdh5/parcels/5.6.0/manifest.json ./
[...]

# 查看下载完成后目录文件
> ll /Archive/Cloudera-Enterprise/cdh5/parcels/5.6.0/
total 1423276
-rwxr-xr-x. 1 root root 1457371397 Nov 23 10:26 CDH-5.6.0-1.cdh5.6.0.p0.45-el6.parcel
-rwxr-xr-x. 1 root root         41 Feb 24  2016 CDH-5.6.0-1.cdh5.6.0.p0.45-el6.parcel.sha1
-rwxr-xr-x. 1 root root      49913 Feb 24  2016 manifest.json
```

##### `Cloudera Enterprise - CM`

- 1. 下载文件

```bash
# 进入目录
> cd /Archive/
 
# 执行下载命令
# 大约需要下载 `700MB` 左右的数据，若时间过久可以通过后台执行方式下载。
# 后台下载命令（ nohup wget -c -r -np -k -L -p https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.6.0/ & ）
> wget -c -r -np -k -L -p https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.6.0/
[...]

# 等待软件下载完成
[...]

# 删除多余文件
> rm -rf `find /Archive/archive.cloudera.com/ -name "index.html*"`
 
# 查看下载主要文件
> ll /Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.6.0/RPMS/x86_64/
total 698960
-rwxr-xr-x 1 root root   4900696 Nov 25 14:58 cloudera-manager-agent-5.6.0-1.cm560.p0.54.el6.x86_64.rpm
-rwxr-xr-x 1 root root 496577388 Nov 25 14:58 cloudera-manager-daemons-5.6.0-1.cm560.p0.54.el6.x86_64.rpm
-rwxr-xr-x 1 root root      8388 Nov 25 14:58 cloudera-manager-server-5.6.0-1.cm560.p0.54.el6.x86_64.rpm
-rwxr-xr-x 1 root root     10128 Nov 25 14:58 cloudera-manager-server-db-2-5.6.0-1.cm560.p0.54.el6.x86_64.rpm
-rwxr-xr-x 1 root root    980296 Nov 25 14:58 enterprise-debuginfo-5.6.0-1.cm560.p0.54.el6.x86_64.rpm
-rwxr-xr-x 1 root root  71204325 Nov 25 14:58 jdk-6u31-linux-amd64.rpm
-rwxr-xr-x 1 root root 142039186 Nov 25 14:58 oracle-j2sdk1.7-1.7.0+update67-1.x86_64.rpm

# 下载 `repo` 文件
> cd /Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/
> wget https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo ./
[...]

# 下载 `RPM-GPG-KEY` 文件
> cd /Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/
> wget https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera ./
[...]

# 查看下载结果
> ll /Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/
total 12
drwxr-xr-x 4 root root 4096 Nov 25 14:58 5.6.0
-rw-r--r-- 1 root root  290 Nov 14 23:04 cloudera-manager.repo
-rw-r--r-- 1 root root 1690 Oct 28  2013 RPM-GPG-KEY-cloudera
```

- 2. 创建目录

```bash
> mkdir -p /Archive/Cloudera-Enterprise
```

- 3. 拷贝文件到创建目录中

```bash
# 执行拷贝命令
> cp -r /Archive/archive.cloudera.com/cm5 /Archive/Cloudera-Enterprise/
 
# 查看拷贝结果
> ll /Archive/Cloudera-Enterprise/
total 8
drwxr-xr-x. 3 root root 4096 Nov 25 11:45 cdh5
drwxr-xr-x. 3 root root 4096 Nov 25 14:54 cm5
```

- 4. 创建软连接

```bash
> ln -s /Archive/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/cloudera-manager.repo /Archive/Cloudera-Enterprise/cm5.repo

# 查看结果
> ll /Archive/Cloudera-Enterprise/
total 12
drwxr-xr-x. 3 root root 4096 Nov 25 11:45 cdh5
drwxr-xr-x. 3 root root 4096 Nov 25 14:54 cm5
lrwxrwxrwx  1 root root   73 Nov 25 15:11 cm5.repo -> /Archive/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
```

- 5. 修改 `repo` 文件内容

```bash
# 将文件中所有 "https://archive.cloudera.com" 替换为 "http://archive.centrin.com.cn/Cloudera-Enterprise"
# 修改后文件内容如下：
> cat /Archive/Cloudera-Enterprise/cm5.repo
```

```bash
[cloudera-manager]
# Packages for Cloudera Manager, Version 5, on RedHat or CentOS 6 x86_64                  
name=Cloudera Manager
baseurl=http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/5.6.0/
gpgkey =http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera    
gpgcheck = 1
```

- 6. 和原始 `repo` 文件对比

```bash
> git diff /Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo /Archive/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/cloudera-manager.repo 

diff --git a/Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo b/Archive/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/c
old mode 100644
new mode 100755
index 28135ba..f8b3a08
--- a/Archive/archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
+++ b/Archive/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
@@ -1,7 +1,7 @@
 [cloudera-manager]
 # Packages for Cloudera Manager, Version 5, on RedHat or CentOS 6 x86_64                 
 name=Cloudera Manager
-baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5/
-gpgkey =https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera    
+baseurl=http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/5.6.0/
+gpgkey =http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera    
 gpgcheck = 1
```

- 7. 删除原始文件

```bash
# 执行删除命令
> rm -rf /Archive/archive.cloudera.com/

# 检查是否已经删除
> ll /Archive/archive.cloudera.com/
ls: cannot access /Archive/archive.cloudera.com/: No such file or directory
```

##### `Scripts`

> **注意：** 本文中使用的脚本基本都是 `internal tools`，主要是用如下三个脚本：
> - `pushssh.sh`: 创建信任关系
> - `pssh.sh`: 多服务器批量执行命令
> - `pscp.sh`: 多服务器批量拷贝文件
> 
> **若没有这些脚本可以去互联网上查找类似脚本**

```bash
# 创建目录
> mkdir -p /Archive/Scripts
> ll -d /Archive/Scripts/
drwxr-xr-x 2 root root 4096 Nov 24 17:55 /Archive/Scripts/

# 从其他服务器上获取 `scripts.zip` 到 /Archive/Scripts/ 目录下
[...]

# 获取后查看结果
> ll /Archive/Scripts/scripts.zip 
-rwxr-xr-x 1 root root 71627 Nov 24 17:55 /Archive/Scripts/scripts.zip
```

##### `Resources`

- 1. 创建目录

```bash
# 创建目录
> mkdir -p /Archive/Resources

# 查看目录
> ll -d /Archive/Resources/
drwxr-xr-x 2 root root 4096 Nov 25 09:20 /Archive/Resources/

# 进入目录
> cd /Archive/Resources/
```

- 2. 下载 `mysql-jdbc-driver` 文件

> **注意：** `mysql-jdbc-driver` 是 `Cloudera Manager`, `Hive`, `oozie`, `...` 等连接 `mysql` 数据库时使用。
> 
> **官网说明：** http://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_ig_mysql.html#cmig_topic_5_5_3
 
```bash
# 进入目录
> cd /Archive/Resources/

# 下载文件
> wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.40.tar.gz ./
[...]

# 查看下载文件
> ll /Archive/Resources/mysql-connector-java-5.1.40.tar.gz 
-rwxr-xr-x 1 root root 3911557 Sep 25 00:35 /Archive/Resources/mysql-connector-java-5.1.40.tar.gz
```

- 3. 创建通过 `http` 方式搭建 `repo` 服务器的 `apache httpd` 配置文件

```bash
# 进入目录
> cd /Archive/Resources/

# 创建 `archive_centrin.conf` 文件，内容如下：
> cat archive_centrin.conf 
```

```bash 
# NameVirtualHost *:80
# do not allow override of this value for the UI's Vhost as it should always be off when generating non-html content such as dynamic images
<VirtualHost *:80>
        DocumentRoot "/Archive"
        ServerName archive.centrin.com.cn
        ErrorLog /var/log/httpd/centrin_archive_error.log

        <Directory "/Archive">
                Options +Indexes 
                AllowOverride All
                Order deny,allow
                Allow from All
        </Directory>
</VirtualHost>
```

- 4. 查看 `Resources` 目录全部内容 

```bash
> ll /Archive/Resources/
total 3824
-rwxr-xr-x 1 root root     424 Nov 23 16:38 archive_centrin.conf
-rwxr-xr-x 1 root root 3911557 Sep 25 00:35 mysql-connector-java-5.1.40.tar.gz
```

#### 通过 `http` 方式搭建软件服务器（ `Repository Server` ）

> **注意：** 本文中使用 `Web Server` 为 `apache httpd`，若想使用其他 `Web Server` 请自行解决。
 
- 1. 查看 `httpd` 服务状态

```bash
> service httpd status
httpd is stopped
```

- 2. 停止 `httpd` 服务

```bash
# 若 `httpd` 是启动状态，需要先停止服务
> service httpd stop
Stopping httpd:                                            [  OK  ]
```

- 3. 拷贝 `apache httpd` 配置文件到 `conf.d` 目录中

```bash
> cp /Archive/Resources/archive_centrin.conf /etc/httpd/conf.d/

# 查看拷贝文件
> ll /etc/httpd/conf.d/archive_centrin.conf
-rw-r--r-- 1 root root 424 Nov 23 15:44 /etc/httpd/conf.d/archive_centrin.conf
```

- 4. 删除 `冲突` 配置文件

> **注意：** 因为 `httpd` 原有的 `welcome.conf` 和 `noindex.html` 文件会导致 `浏览文件` 和 `软连接` 功能失效，所以需要删除这两个文件。**这里采用逻辑删除方式（改名）**

- 4.1. `"删除"` `welcome.conf` 文件

```bash
# 进入目录
> cd /etc/httpd/conf.d/

# 删除文件
> mv welcome.conf welcome.conf.deleted
 
# 检查
> ll
total 28
-rw-r--r--  1 root root  424 Nov 23 15:44 archive_centrin.conf
-rw-r--r--. 1 root root  118 May 20  2009 mod_dnssd.conf
-rw-r--r--. 1 root root  392 Aug  2  2013 README
-rw-r--r--  1 root root 9473 Aug  2  2013 ssl.conf
-rw-r--r--. 1 root root  299 Aug  2  2013 welcome.conf.deleted 
```

- 4.2. `"删除"` `noindex.html` 文件

```bash
# 进入目录
> cd /var/www/error/

# 删除文件
> mv noindex.html noindex.html.deleted

# 检查
> ll
total 196
[...]
-rw-r--r--. 1 root root  3985 Aug  2  2013 noindex.html.deleted
[...]
```

- 5. 启动 `httpd` 服务

```bash
# 启动服务
> service httpd start
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.3.49 for ServerName
                                                           [  OK  ]
```

- 6. 确认 `httpd` 状态

```bash
# 查看启动状态                                                         
> service httpd status
httpd (pid  19397) is running...
```

- 7. 关闭防火墙

```bash
# 查看防火墙状态
> service iptables status
Table: filter
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
2    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
5    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         
1    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         

# 关闭防火墙
> service iptables stop
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]

# 设置重启不自动启动
> chkconfig iptables off

# 检查 `[2-5]` 项均为 `off`
> chkconfig --list iptables
iptables        0:off   1:off   2:off   3:off   4:off   5:off   6:off

# 查看防火墙状态
> service iptables status
iptables: Firewall is not running.
```

- 8. 登陆页面查看启动情况

> **注意：** 若有 `403` 等访问权限错误，可能还需要关闭 `SELinux` 。具体关闭 `SELinux` 方式可以详见下文中的 `关闭SELinux` 章节。
> 
> 一定要保证浏览器访问 `http://10.0.3.49/` 地址的时候和下图中显示的一样。

通过浏览器访问地址：http://10.0.3.49/

![](media/14797132371800/14800621605311.jpg)

#### 设置软件服务器（ `Repository Server` ）到 `/etc/hosts` 文件中

```bash
# 若成功执行该命令后，无任何输出，则说明没有设置软件服务器到`/etc/hosts`文件中
> cat /etc/hosts | grep archive.centrin.com.cn

# 执行设置命令（**注意**该命令会在`/etc/hosts`文件后面添加一行，不可重复执行）
> echo "10.0.3.49 archive.centrin.com.cn" >> /etc/hosts

# 若成功执行该行命令后，在出现的输出结果中确定`ip`地址是否是你搭建软件服务器的`ip`
> cat /etc/hosts | grep archive.centrin.com.cn
10.0.3.49 archive.centrin.com.cn

# 检查是否可以`ping`通软件服务器
> ping archive.centrin.com.cn
PING archive.centrin.com.cn (10.0.3.49) 56(84) bytes of data.
64 bytes from archive.centrin.com.cn (10.0.3.49): icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from archive.centrin.com.cn (10.0.3.49): icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from archive.centrin.com.cn (10.0.3.49): icmp_seq=3 ttl=64 time=0.066 ms
[...]
```

#### 验证软件服务器（ `Repository Server` ）

- 1. 删除原有 `repo` 文件

> **注意：** 若不确定 `/etc/yum.repos.d/` 目录下是否所有的文件均可以删除，则需要提前备份该目录下文件，以便删除后找不回来。
> 
> **为了下文中 `同步软件服务器（ Repository Server ）repo 文件` 章节方便，需要 `依次登陆集群所有的服务器` 执行 "删除原有 `repo` 文件" 操作**

```bash
# 依次登陆集群中所有的服务器，执行删除
> rm -rf /etc/yum.repos.d/*

# 检查
> ll /etc/yum.repos.d/
total 0
```

- 2. 下载 `repo` 文件

> **注意：** 该操作只需在 `repo` 服务器上执行即可。

```bash
# 进入目录
> cd /etc/yum.repos.d/

# 下载 `rhel - repo` 文件 
> wget http://10.0.3.49/RedHat-Enterprise-Linux/rhel-6.5-media.repo ./
 
# 下载 `cm5 - repo` 文件
> wget http://10.0.3.49/Cloudera-Enterprise/cm5.repo ./

# 检查
> ll
total 8
-rw-r--r-- 1 root root 336 Nov 23 17:30 cm5.repo
-rw-r--r-- 1 root root 227 Nov 23 16:14 rhel-6.5-media.repo 
```

- 3. 清除缓存

```bash
> yum clean all
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Cleaning repos: cloudera-manager rhel-media
Cleaning up Everything
```

- 4. 查看 `repolist`

```bash
> yum repolist

Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
cloudera-manager                                                                                                      |  951 B     00:00     
cloudera-manager/primary                                                                                              | 4.1 kB     00:00     
cloudera-manager                                                                                                                         7/7
rhel-media                                                                                                            | 3.9 kB     00:00     
rhel-media/primary_db                                                                                                 | 3.1 MB     00:00     
repo id                                                      repo name                                                                 status
cloudera-manager                                             Cloudera Manager                                                              7
rhel-media                                                   Red Hat Enterprise Linux 6.5                                              3,690
repolist: 3,697
```

> **注意：** 从执行结果可以看出现在一共有 `3,697` 个软件。
> - `cloudera-manager` 提供其中 `7` 个。
> - `rhel-media` 提供其中 `3,690` 个。

### 设置主机名（ `hosts` ）

#### 设置集群服务器（ `Cluster Server` ）到 `/etc/hosts` 文件中

> 将所有需要使用的服务器`ip`和`host`写入`/etc/hosts`文件中。格式如下例：

```bash
> cat /etc/hosts
```

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354
```

#### 设置软件服务器（ `Repository Server` ）到 `/etc/hosts` 文件中

```bash
> cat /etc/hosts
```

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
```

#### 设置时间服务器（ `NTP Server` ）到 `/etc/hosts` 文件中

```bash
> cat /etc/hosts
```

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
```

#### 主机名文件（`/etc/hosts`）顺序特别注意

> **!!! 注意：若有重复的主机名（`HOST`），比如本例中`10.0.3.49`同时为（`server349`, `archive.centrin.com.cn`, `ntp.centrin.com.cn`）那么最后的顺序一定是`server349`在最上面，同时也就是`archive.centrin.com.cn`和`ntp.centrin.com.cn`一定要在最下面，否则在会面会导致安装不成功。**

### 设置软件服务器（ `Repository Server` ）

#### 下载 `repo` 文件

> **注意：** 因为在 `验证软件服务器（ Repository Server ）` 章节已经将 `repo` 文件下载到 `/etc/yum.repo.d/` 目录中；所以在这里不再赘述。
> 
> 可根据下面命令验证 `repo` 文件是否已经下载完成。若和执行结果不一致请查看 `验证软件服务器（ Repository Server ）` 章节。

```bash
# 验证 `repo` 文件是否已经下载；若执行结果和下文中不一致，请详见 `验证软件服务器（ Repository Server ）` 章节
> cd /etc/yum.repos.d/
> ll
total 8
-rw-r--r-- 1 root root 336 Nov 23 17:30 cm5.repo
-rw-r--r-- 1 root root 227 Nov 23 16:14 rhel-6.5-media.repo
```

### 关闭 `SELinux`

> **注意：** `/etc/hosts` 文件中配置的所有服务器均需要关闭 `SELinux`。

#### 备份当前 `SELinux` 配置文件

```bash
> cp /etc/selinux/config /etc/selinux/config.raw
```

#### 修改 `/etc/selinux/config` 文件

```bash
# 将 SELINUX=enforcing 修改为 SELINUX=disabled
> vim /etc/selinux/config
```

#### 查看修改后内容

```bash
> cat /etc/selinux/config
```

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```

#### 和原始文件对比修改内容

> **注意：** `git diff` 命令需安装 `git` ( `yum -y install git` )。

```bash
> git diff /etc/selinux/config.raw /etc/selinux/config
diff --git a/etc/selinux/config.raw b/etc/selinux/config
index 3a77ea4..ff7b8ac 100644
--- a/etc/selinux/config.raw
+++ b/etc/selinux/config
@@ -4,7 +4,7 @@
 #     enforcing - SELinux security policy is enforced.
 #     permissive - SELinux prints warnings instead of enforcing.
 #     disabled - No SELinux policy is loaded.
-SELINUX=enforcing
+SELINUX=disabled
 # SELINUXTYPE= can take one of these two values:
 #     targeted - Targeted processes are protected,
 #     mls - Multi Level Security protection.
```

#### 重启服务器

```bash
> reboot

Broadcast message from root@server349
        (/dev/pts/0) at 16:23 ...

The system is going down for reboot NOW!
```

#### 查看 `SELinux` 状态

```bash
> /usr/sbin/sestatus -v
SELinux status:                 disabled
```

### 获取使用脚本 （ `internal tools` ）

#### 通过软件服务器（ `Repository Server` ）下载使用脚本（ `internal tools` ）

```bash
# 进入目录
> cd /opt/
 
# 下载使用脚本
> wget http://archive.centrin.com.cn/Scripts/scripts.zip ./
[...]

# 检查
> ll 
-rw-r--r-- 1 root root 71627 Nov 24 17:55 /opt/scripts.zip
```

#### 解压

```bash
# 执行解压
> cd /opt/
> unzip scripts.zip
[...]

# 检查
> ll /opt/scripts/
[...]
-rw-r--r-- 1 root root   105 Jan 26  2015 allhosts    # 配置文件
[...]
-rwxr-xr-x 1 root root   704 Jan 26  2015 pscp.sh     # 多服务器批量拷贝文件脚本
[...]
-rwxr-xr-x 1 root root   873 Jan 26  2015 pssh.sh     # 多服务器批量执行命令脚本
-rwxr-xr-x 1 root root   600 Jan 26  2015 pushssh.sh  # 创建信任关系脚本
[...]
```

#### 修改配置文件

> 将 `集群服务器` 列表写入配置文件。

```bash
# 删除现有配置文件
> rm -rf /opt/scripts/allhosts

# 将如下内容写入 `/opt/scripts/allhosts` 文件
> cat /opt/scripts/allhosts 
``` 

```bash
server349
server351
server352
server353
server354
```

#### 删除脚本包

> 删除原有下载 `.zip` 的脚本包

```bash
# 执行删除
> rm -rf /opt/scripts.zip
 
# 检查
> ll /opt/scripts.zip
ls: cannot access /opt/scripts.zip: No such file or directory
```

### 创建信任关系

#### 通过 `internal tools` - `/opt/scripts/pushssh.sh` 创建信任关系

> **注意：** 需要和 `/etc/hosts` 文件中设置的 `所有服务器` 依次创建信息关系。

```bash
> /opt/scripts/pushssh.sh root@server349
[...]

> /opt/scripts/pushssh.sh root@server351
[...]

> /opt/scripts/pushssh.sh root@server352
[...]

> /opt/scripts/pushssh.sh root@server353
[...]

> /opt/scripts/pushssh.sh root@server354
[...]
```

> **注意：** 解决建议信任关系系未知问题
> 1. `~/.ssh`目录的权限应是`700`。
> 2. `~/.ssh/authorized_keys`文件的权限应是`644`。
> 3. 重新建立信任关系。

#### 检查

> **注意：** 依次确认创建信任关系是否成功。

```bash
> ssh server349
Last login: Wed Nov 23 10:20:40 2016 from server349
> exit
logout
Connection to server349 closed.

> ssh server351
Last login: Wed Nov 23 10:21:05 2016 from 10.0.3.49
> exit
logout
Connection to server351 closed.

> ssh server352
Last login: Wed Nov 23 10:21:20 2016 from 10.0.3.49
> exit
logout
Connection to server352 closed.

> ssh server353
Last login: Wed Nov 23 10:21:31 2016 from 10.0.3.49
> exit
logout
Connection to server353 closed.

> ssh server354
Last login: Wed Nov 23 10:21:47 2016 from 10.0.3.49
> exit
logout
Connection to server354 closed.
```

### 同步主机名文件（ `/etc/hosts` ）

> **!!! 特别注意`/etc/hosts`文件设置顺序，具体请详见`主机名文件（/etc/hosts）顺序特别注意`章节。**

#### 通过 `internal tools` - `/opt/scripts/pscp.sh` 同步主机名文件（ `/etc/hosts` ）到所有集群服务器中

```bash
> /opt/scripts/pscp.sh /opt/scripts/allhosts /etc/hosts /etc/hosts

/opt/scripts/pscp.sh: line 11: [: /etc/hosts: unary operator expected
Source files to SCP: /etc/hosts



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/hosts root@server349:/etc/hosts
=========================================================
hosts                                                                                                         100%  293     0.3KB/s   00:00    
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/hosts root@server351:/etc/hosts
=========================================================
hosts                                                                                                         100%  293     0.3KB/s   00:00    
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/hosts root@server352:/etc/hosts
=========================================================
hosts                                                                                                         100%  293     0.3KB/s   00:00    
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/hosts root@server353:/etc/hosts
=========================================================
hosts                                                                                                         100%  293     0.3KB/s   00:00    
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/hosts root@server354:/etc/hosts
=========================================================
hosts                                                                                                         100%  293     0.3KB/s   00:00    
====================End==================================
```

#### 通过 `internal tools` - `/opt/scripts/pssh.sh` 检查同步结果

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts 'cat /etc/hosts'

CMD to Exec: cat /etc/hosts



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "cat /etc/hosts"
=========================================================
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "cat /etc/hosts"
=========================================================
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "cat /etc/hosts"
=========================================================
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "cat /etc/hosts"
=========================================================
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "cat /etc/hosts"
=========================================================
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.3.49 server349
10.0.3.51 server351
10.0.3.52 server352
10.0.3.53 server353
10.0.3.54 server354

10.0.3.49 archive.centrin.com.cn
10.0.3.49 ntp.centrin.com.cn
====================End==================================




===============================================================================
================= Script Finished Totally in 1 seconds ============================
===============================================================================
```


### 同步软件服务器（ `Repository Server` ）`repo` 文件

#### 通过 internal tools - /opt/scripts/pscp.sh 同步 `repo` 文件

```bash
> /opt/scripts/pscp.sh /opt/scripts/allhosts /etc/yum.repos.d/ /etc/

/opt/scripts/pscp.sh: line 11: [: /etc/: unary operator expected
Source files to SCP: /etc/yum.repos.d/



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/yum.repos.d/ root@server349:/etc/
=========================================================
cm5.repo                                                                        100%  336     0.3KB/s   00:00    
rhel-6.5-media.repo                                                             100%  227     0.2KB/s   00:00    
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/yum.repos.d/ root@server351:/etc/
=========================================================
cm5.repo                                                                        100%  336     0.3KB/s   00:00    
rhel-6.5-media.repo                                                             100%  227     0.2KB/s   00:00    
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/yum.repos.d/ root@server352:/etc/
=========================================================
cm5.repo                                                                        100%  336     0.3KB/s   00:00    
rhel-6.5-media.repo                                                             100%  227     0.2KB/s   00:00    
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/yum.repos.d/ root@server353:/etc/
=========================================================
cm5.repo                                                                        100%  336     0.3KB/s   00:00    
rhel-6.5-media.repo                                                             100%  227     0.2KB/s   00:00    
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: scp -r /etc/yum.repos.d/ root@server354:/etc/
=========================================================
cm5.repo                                                                        100%  336     0.3KB/s   00:00    
rhel-6.5-media.repo                                                             100%  227     0.2KB/s   00:00    
====================End================================== 
```
 
#### 通过 `internal tools` - `/opt/scripts/pssh.sh` 检查同步结果

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "ls -l /etc/yum.repos.d/"

CMD to Exec: ls -l /etc/yum.repos.d/



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "ls -l /etc/yum.repos.d/"
=========================================================
total 8
-rw-r--r-- 1 root root 336 Nov 28 15:16 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28 15:16 rhel-6.5-media.repo
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "ls -l /etc/yum.repos.d/"
=========================================================
total 8
-rw-r--r-- 1 root root 336 Nov 28 15:16 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28 15:16 rhel-6.5-media.repo
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "ls -l /etc/yum.repos.d/"
=========================================================
total 8
-rw-r--r-- 1 root root 336 Nov 28 15:16 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28 15:16 rhel-6.5-media.repo
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "ls -l /etc/yum.repos.d/"
=========================================================
total 8
-rw-r--r-- 1 root root 336 Nov 28 15:16 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28 15:16 rhel-6.5-media.repo
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "ls -l /etc/yum.repos.d/"
=========================================================
total 8
-rw-r--r-- 1 root root 336 Nov 28 15:16 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28 15:16 rhel-6.5-media.repo
====================End==================================




===============================================================================
================= Script Finished Totally in 1 seconds ============================
===============================================================================
```

### 关闭防火墙

#### 通过 `internal tools` - `/opt/scripts/pssh.sh` 执行关闭防火墙

> 命令完成功能：
> - 1. 关闭 `iptables` 防火墙
> - 2. 设置 `iptables` 防火墙重启后不自动启动
> - 3. 关闭 `ip6tables` 防火墙
> - 4. 设置 `ip6tables` 防火墙重启后不自动启动
 
```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"

CMD to Exec: service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"
=========================================================
ip6tables: Setting chains to policy ACCEPT: filter [  OK  ]
ip6tables: Flushing firewall rules: [  OK  ]
ip6tables: Unloading modules: [  OK  ]
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"
=========================================================
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Unloading modules: [  OK  ]
ip6tables: Setting chains to policy ACCEPT: filter [  OK  ]
ip6tables: Flushing firewall rules: [  OK  ]
ip6tables: Unloading modules: [  OK  ]
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"
=========================================================
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Unloading modules: [  OK  ]
ip6tables: Setting chains to policy ACCEPT: filter [  OK  ]
ip6tables: Flushing firewall rules: [  OK  ]
ip6tables: Unloading modules: [  OK  ]
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"
=========================================================
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Unloading modules: [  OK  ]
ip6tables: Setting chains to policy ACCEPT: filter [  OK  ]
ip6tables: Flushing firewall rules: [  OK  ]
ip6tables: Unloading modules: [  OK  ]
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "service iptables stop;chkconfig iptables off;service ip6tables stop;chkconfig ip6tables off;"
=========================================================
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Unloading modules: [  OK  ]
ip6tables: Setting chains to policy ACCEPT: filter [  OK  ]
ip6tables: Flushing firewall rules: [  OK  ]
ip6tables: Unloading modules: [  OK  ]
====================End==================================




===============================================================================
================= Script Finished Totally in 4 seconds ============================
===============================================================================
```

#### 通过 `internal tools` - `/opt/scripts/pssh.sh` 检查防火墙关闭情况
 
```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "service iptables status;service ip6tables status;"

CMD to Exec: service iptables status;service ip6tables status;



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "service iptables status;service ip6tables status;"
=========================================================
iptables: Firewall is not running.
ip6tables: Firewall is not running.
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "service iptables status;service ip6tables status;"
=========================================================
iptables: Firewall is not running.
ip6tables: Firewall is not running.
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "service iptables status;service ip6tables status;"
=========================================================
iptables: Firewall is not running.
ip6tables: Firewall is not running.
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "service iptables status;service ip6tables status;"
=========================================================
iptables: Firewall is not running.
ip6tables: Firewall is not running.
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "service iptables status;service ip6tables status;"
=========================================================
iptables: Firewall is not running.
ip6tables: Firewall is not running.
====================End==================================




===============================================================================
================= Script Finished Totally in 1 seconds ============================
===============================================================================
```

### 搭建时间服务器（ `NTP Server` ）

#### 安装 `ntp` 

> **注意：** 一般服务器在安装操作系统的同时已经将 `ntp` 安装。

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "yum -y install ntp"
 
CMD to Exec: yum -y install ntp



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "yum -y install ntp"
=========================================================
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Package ntp-4.2.6p5-1.el6.x86_64 already installed and latest version
Nothing to do
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "yum -y install ntp"
=========================================================
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Package ntp-4.2.6p5-1.el6.x86_64 already installed and latest version
Nothing to do
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "yum -y install ntp"
=========================================================
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Package ntp-4.2.6p5-1.el6.x86_64 already installed and latest version
Nothing to do
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "yum -y install ntp"
=========================================================
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Package ntp-4.2.6p5-1.el6.x86_64 already installed and latest version
Nothing to do
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "yum -y install ntp"
=========================================================
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Package ntp-4.2.6p5-1.el6.x86_64 already installed and latest version
Nothing to do
====================End==================================




===============================================================================
================= Script Finished Totally in 3 seconds ============================
===============================================================================
```

#### 设置 `ntp` 开机自启动

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "chkconfig ntpd on"
 
CMD to Exec: chkconfig ntpd on



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "chkconfig ntpd on"
=========================================================
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "chkconfig ntpd on"
=========================================================
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "chkconfig ntpd on"
=========================================================
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "chkconfig ntpd on"
=========================================================
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "chkconfig ntpd on"
=========================================================
====================End==================================




===============================================================================
================= Script Finished Totally in 0 seconds ============================
===============================================================================
```

#### 备份 `ntp` 配置文件

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "cp /etc/ntp.conf /etc/ntp.conf.raw"

CMD to Exec: cp /etc/ntp.conf /etc/ntp.conf.raw



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "cp /etc/ntp.conf /etc/ntp.conf.raw"
=========================================================
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "cp /etc/ntp.conf /etc/ntp.conf.raw"
=========================================================
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "cp /etc/ntp.conf /etc/ntp.conf.raw"
=========================================================
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "cp /etc/ntp.conf /etc/ntp.conf.raw"
=========================================================
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "cp /etc/ntp.conf /etc/ntp.conf.raw"
=========================================================
====================End==================================




===============================================================================
================= Script Finished Totally in 0 seconds ============================
===============================================================================
```

#### 修改 `主` 节点配置文件

> **注意：** 下文贴出需要修改内容

```bash
> cat /etc/ntp.conf
```

```bash
[...]

# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server 127.127.1.0
fudge 127.127.1.0 stratum 8 

[...]
```

#### 对比 `主` 节点配置文件

```bash
> git diff /etc/ntp.conf.raw /etc/ntp.conf

diff --git a/etc/ntp.conf.raw b/etc/ntp.conf
index e18f59e..8ad2bd7 100644
--- a/etc/ntp.conf.raw
+++ b/etc/ntp.conf
@@ -19,10 +19,12 @@ restrict -6 ::1
 
 # Use public servers from the pool.ntp.org project.
 # Please consider joining the pool (http://www.pool.ntp.org/join.html).
-server 0.rhel.pool.ntp.org iburst
-server 1.rhel.pool.ntp.org iburst
-server 2.rhel.pool.ntp.org iburst
-server 3.rhel.pool.ntp.org iburst
+#server 0.rhel.pool.ntp.org iburst
+#server 1.rhel.pool.ntp.org iburst
+#server 2.rhel.pool.ntp.org iburst
+#server 3.rhel.pool.ntp.org iburst
+server 127.127.1.0
+fudge 127.127.1.0 stratum 8 
 
 #broadcast 192.168.1.255 autokey       # broadcast server
 #broadcastclient                       # broadcast client
```

#### 依次修改 `其他` 节点配置文件

> **注意：** 依次修改除 `主`节点以外的所有节点，具体修改内容详见下文。

```bash
> cat /etc/ntp.conf
```

```bash
[...]

# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server ntp.centrin.com.cn

[...]
```

#### 对比 `其他` 节点配置文件

```bash
> git diff /etc/ntp.conf.raw /etc/ntp.conf
 
diff --git a/etc/ntp.conf.raw b/etc/ntp.conf
index e18f59e..185e5f9 100644
--- a/etc/ntp.conf.raw
+++ b/etc/ntp.conf
@@ -19,10 +19,11 @@ restrict -6 ::1
 
 # Use public servers from the pool.ntp.org project.
 # Please consider joining the pool (http://www.pool.ntp.org/join.html).
-server 0.rhel.pool.ntp.org iburst
-server 1.rhel.pool.ntp.org iburst
-server 2.rhel.pool.ntp.org iburst
-server 3.rhel.pool.ntp.org iburst
+#server 0.rhel.pool.ntp.org iburst
+#server 1.rhel.pool.ntp.org iburst
+#server 2.rhel.pool.ntp.org iburst
+#server 3.rhel.pool.ntp.org iburst
+server ntp.centrin.com.cn
 
 #broadcast 192.168.1.255 autokey       # broadcast server
 #broadcastclient                       # broadcast client
```

#### 重启 `主` 节点 `ntp` 服务

```bash
> service ntpd restart
Shutting down ntpd:                                        [  OK  ]
Starting ntpd:                                             [  OK  ]
```

#### 校验 `主` 节点 `ntp` 服务状态

> **注意：** 如果提示 `No association ID's returned` 错误，将之前需要修改的内容手动打入文件中，而不是使用 `复制` / `粘贴`。然后重启等一会再查看结果。
> 
> **注意：** 验证 `remote` 栏中是 `LOCAL(0)` ，表示使用本地节点。

```bash
> ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*LOCAL(0)        .LOCL.           5 l   17   64    1    0.000    0.000   0.000
```

> **注意：**  也可以使用 `ntpdate` 验证 `ntp` 服务器 `主` 节点是否搭建成功。若提示没有 `ntpdate` 命令，可以通过 `yum -y install ntpdate` 安装。
 
```bash
> ntpdate -d ntp.centrin.com.cn

29 Nov 09:43:18 ntpdate[32583]: ntpdate 4.2.6p5@1.2349-o Mon Jul 15 09:22:50 UTC 2013 (1)
Looking for host ntp.centrin.com.cn and service ntp
host found : server349
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
server 10.0.3.49, port 123
stratum 6, precision -23, leap 00, trust 000
refid [10.0.3.49], delay 0.02562, dispersion 0.00000
transmitted 4, in filter 4
reference time:    dbe75b7c.1b31ae33  Tue, Nov 29 2016  9:42:20.106
originate timestamp: dbe75bb6.ad869791  Tue, Nov 29 2016  9:43:18.677
transmit timestamp:  dbe75bb6.ad85a3cc  Tue, Nov 29 2016  9:43:18.677
filter delay:  0.02568  0.02563  0.02562  0.02562 
         0.00000  0.00000  0.00000  0.00000 
filter offset: -0.00001 -0.00000 -0.00000 -0.00000
         0.000000 0.000000 0.000000 0.000000
delay 0.02562, dispersion 0.00000
offset -0.000003

29 Nov 09:43:18 ntpdate[32583]: adjust time server 10.0.3.49 offset -0.000003 sec
```

#### 重启 `其他` 节点 `ntp` 服务

```bash
> service ntpd restart
Shutting down ntpd:                                        [  OK  ]
Starting ntpd:                                             [  OK  ]
```

#### 校验 `其他` 节点 `ntp` 服务状态

> **注意：** 验证 `remote` 栏中是 `主` 节点的主机名（ `hosts` ）

```bash
> ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 server349       LOCAL(0)         6 u   23   64  177    0.099  -13.431   0.034
```

> **注意：**  也可以使用 `ntpdate` 验证 `ntp` 服务器 `主` 节点是否搭建成功。若提示没有 `ntpdate` 命令，可以通过 `yum -y install ntpdate` 安装。
 
```bash
> ntpdate -d ntp.centrin.com.cn

29 Nov 09:47:42 ntpdate[33710]: ntpdate 4.2.6p5@1.2349-o Mon Jul 15 09:22:50 UTC 2013 (1)
Looking for host ntp.centrin.com.cn and service ntp
host found : server349
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
transmit(10.0.3.49)
receive(10.0.3.49)
server 10.0.3.49, port 123
stratum 6, precision -23, leap 00, trust 000
refid [10.0.3.49], delay 0.02565, dispersion 0.00000
transmitted 4, in filter 4
reference time:    dbe75cbc.204da8b9  Tue, Nov 29 2016  9:47:40.126
originate timestamp: dbe75cbe.5ac44cee  Tue, Nov 29 2016  9:47:42.354
transmit timestamp:  dbe75cbe.5ad75d81  Tue, Nov 29 2016  9:47:42.354
filter delay:  0.02573  0.02565  0.02565  0.02565 
         0.00000  0.00000  0.00000  0.00000 
filter offset: -0.00034 -0.00032 -0.00032 -0.00032
         0.000000 0.000000 0.000000 0.000000
delay 0.02565, dispersion 0.00000
offset -0.000325

29 Nov 09:47:42 ntpdate[33710]: adjust time server 10.0.3.49 offset -0.000325 sec
```

#### 整体校验

> **注意：** 登陆 `主` 节点执行如下命令。
> 若发现执行命令后时间还不一致的话，请查看各个服务的`防火墙`和`SELinux`是否已经关闭。
> 若还是不行请重新一下操作系统试试。

```bash
> /opt/scripts/pssh.sh /opt/scripts/allhosts "date"

CMD to Exec: date



--------------- server349  #1/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server349 "date"
=========================================================
Mon Nov 28 18:01:58 CST 2016
====================End==================================



--------------- server351  #2/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server351 "date"
=========================================================
Mon Nov 28 18:01:58 CST 2016
====================End==================================



--------------- server352  #3/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server352 "date"
=========================================================
Mon Nov 28 18:01:58 CST 2016
====================End==================================



--------------- server353  #4/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server353 "date"
=========================================================
Mon Nov 28 18:01:58 CST 2016
====================End==================================



--------------- server354  #5/5 /opt/scripts/allhosts ---------------
Command to Exec: ssh root@server354 "date"
=========================================================
Mon Nov 28 18:01:59 CST 2016
====================End==================================




===============================================================================
================= Script Finished Totally in 1 seconds ============================
===============================================================================
```

### 搭建外部数据库（ `External Databases` - `mysql` ）

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
log_bin=/var/lib/mysql/mysql_binary_log

# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
binlog_format = mixed

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

#### 安装 `mysql` 驱动（`JDBC`）

```bash
# 下载驱动
> cd /opt/
> wget http://archive.centrin.com.cn/Resources/mysql-connector-java-5.1.40.tar.gz ./
> ll 
[...]
-rw-r--r--  1 root root 3911557 Sep 25 00:35 mysql-connector-java-5.1.40.tar.gz
[...] 

# 解压驱动
> tar -zxvf mysql-connector-java-5.1.40.tar.gz 
[...]
> ll mysql-connector-java-5.1.40
-rw-r--r-- 1 root root  90699 Sep 25 02:35 build.xml
-rw-r--r-- 1 root root 241467 Sep 25 02:35 CHANGES
-rw-r--r-- 1 root root  18122 Sep 25 02:35 COPYING
drwxr-xr-x 2 root root   4096 Nov 23 18:30 docs
-rw-r--r-- 1 root root 990927 Sep 25 02:35 mysql-connector-java-5.1.40-bin.jar
-rw-r--r-- 1 root root  61407 Sep 25 02:35 README
-rw-r--r-- 1 root root  63658 Sep 25 02:35 README.txt
drwxr-xr-x 8 root root   4096 Sep 25 02:35 src

# 创建目录
> mkdir -p /usr/share/java/

# 将驱动文件移动到目录中
> cp /opt/mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar /usr/share/java/mysql-connector-java.jar
> ll /usr/share/java/mysql-connector-java.jar
-rw-r--r-- 1 root root 990927 Nov 23 18:32 /usr/share/java/mysql-connector-java.jar

# 删除驱动
> cd /opt/
> rm -rf mysql-connector-java-5.1.40*
```

#### 初始化 `Activity Monitor`, `Hive Metastore Server`, `Oozie Server` 数据库

```bash
# 登陆数据库
> mysql -uroot -proot123

# 创建 `Activity Monitor` 使用数据库
mysql> create database amon DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

# 创建 `Hive Metastore Server` 使用数据库
mysql> create database metastore DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

# 创建 `Oozie Server` 使用数据库
mysql> create database oozie DEFAULT CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)

# 查看已创建的数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| amon               |
| metastore          |
| mysql              |
| oozie              |
| scm                |
+--------------------+
6 rows in set (0.00 sec)

# 赋权
mysql> grant all on *.* TO 'root'@'%' IDENTIFIED BY 'root123';
Query OK, 0 rows affected (0.00 sec)

# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 退出 `mysql cli` 模式
mysql> exit;
Bye
```

## 安装 `Cloudera Manager`

### 安装 `Oracle JDK`

```bash
> yum -y install oracle-j2sdk1.7
```

### 安装 `Cloudera Manager Server` 和 `Cloudera Manager Daemons`

```bash
> yum -y install cloudera-manager-daemons cloudera-manager-server
```

### 初始化 `Cloudera Manager` 需要使用的数据库

```bash
# 执行 `Cloudera Manager` 提供的脚本初始化 `Cloudera Manager` 需要使用的数据库 `scm`
> /usr/share/cmf/schema/scm_prepare_database.sh mysql -uroot -proot123 --scm-host localhost scm scm scm

JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
Verifying that we can write to /etc/cloudera-scm-server
Creating SCM configuration file in /etc/cloudera-scm-server
Executing:  /usr/java/jdk1.7.0_67-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/cmf/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
```

### 启动 `Cloudera Manager Server`

> **注意：** 第一次启动需要一些时间，因为 `Cloudera Manager Server` 需要连接到 `mysql` 数据库建立一些要使用的数据库表。
> 可以查看日志（ `/var/log/cloudera-scm-server/cloudera-scm-server.log` ）文件，来确认 `Cloudera Manager Server` 是否启动完成或者失败。
> 启动成功后会

```bash
> service cloudera-scm-server restart
cloudera-scm-server is already stopped
Starting cloudera-scm-server:                              [  OK  ] 
```

## 通过 `Cloudera Manager` - ` WEB UI` 安装 `CDH`

> **注意：** 进入 `WEB UI` 后根据提示一步步操作即可。
> 第一次启动需要一些时间，因为 `Cloudera Manager Server` 需要连接到 `mysql` 数据库建立一些要使用的数据库表。
> 可以查看日志（ `/var/log/cloudera-scm-server/cloudera-scm-server.log` ）文件，来确认 `Cloudera Manager Server` 是否启动完成或者失败。

---

![](media/14797132371800/14804067951035.jpg)

---

![](media/14797132371800/14804068275699.jpg)

---

![](media/14797132371800/14804068511090.jpg)

---

![](media/14797132371800/14804069205778.jpg)

---

![](media/14797132371800/14804069633039.jpg)

---

![](media/14797132371800/14804069910353.jpg)

---

> **注意**：
> 首先`远程Parcel存储库URL`原本应有很多选项，最好将其他选项删除。
> 输出`URL`地址时，最好输出主机名而不是`IP`地址。
> **注意开头是`http://` 而不是 `https://`**
> `远程Parcel存储库URL`: `http://archive.centrin.com.cn/Cloudera-Enterprise/cdh5/parcels/5.6.0/`

![](media/14797132371800/14804070365657.jpg)

---

> **注意：**
> `自定义存储库`: `http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/5.6.0/`
> `自定义存储库秘钥`: `http://archive.centrin.com.cn/Cloudera-Enterprise/cm5/redhat/6/x86_64/cm/RPM-GPG-KEY-cloudera`

![](media/14797132371800/14869759123643.jpg)

> **!!! 注意检查上图中的所有配置，这步很重要！**

---

> **注意：**因为是新重新再别的服务器上创建的环境截图，所以`IP`地址不同是正常情况。

![](media/14797132371800/14869760349267.jpg)

---

> **特别注意：这步一定不能选单用户模式！！！**

![](media/14797132371800/14872274453218.jpg)

---

![](media/14797132371800/14869752309740.jpg)

---

![](media/14797132371800/14869762917049.jpg)

---

![](media/14797132371800/14804070758467.jpg)

> **注意：** 由于在此页面会停留很长时间，所以简要说明一下每个步骤工作流程。
> 
> - 1. 已下载：将 `parcel` 包从之前配置的 `repo` 地址下载到启动 `Cloudera-Manager Server` 服务器的本地 `/opt/cloudera/parcel-repo` 目录中。
> - 2. 已分配：将 `Cloudera-Manager Server` 服务器的本地 `/opt/cloudera/parcel-repo` 目录中的 `parcel` 包，分配传输到每个节点的 `/opt/cloudera/parcel-cache`  目录中。
> - 3. 已解压：将每个节点 `/opt/cloudera/parcel-cache` 目录中的 `parcel` 包解压到 `/opt/cloudera/parcels` 目录中。
> - 4. 激活后，在 `/opt/cloudera/parcels` 目录中，创建一个 `CDH` 软连接目录，指向 `/opt/cloudera/parcels` 目录中原来解压完成的包。

---

![](media/14797132371800/14804075421113.jpg)

---

![](media/14797132371800/14804084574751.jpg)

---

![](media/14797132371800/14804085436435.jpg)

> **注意：** 有一些问题需要解决，否则可能会导致安装集群时出现错误。

---

> **问题：**
> `Cloudera 建议将 /proc/sys/vm/swappiness 设置为 0。当前设置为 60。
> 使用 sysctl 命令在运行时更改该设置并编辑 /etc/sysctl.conf 以在重启后保存该设置。
> 您可以继续进行安装，但可能会遇到问题，Cloudera Manager 报告您的主机由于交换运行状况不佳。`
>
> **解决：**
> `echo 0 > /proc/sys/vm/swappiness`

---

> **问题：**
> `已启用“透明大页面”，它可能会导致重大的性能问题。版本为“/sys/kernel/mm/transparent_hugepage”且发行版为“{1}”的 Kernel 已将 enabled 设置为“{2}”，并将 defrag 设置为“{3}”。请运行“echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag”以禁用此设置，然后将同一命令添加到一个 init 脚本中，如 /etc/rc.local，这样当系统重启时就会予以设置。或者，升级到 RHEL 6.5 或更新版本，它们不存在此错误。`
>
> **解决：**
> `echo never > /sys/kernel/mm/transparent_hugepage/enabled`
> `echo never > /sys/kernel/mm/transparent_hugepage/defrag`

---

![](media/14797132371800/14869769600570.jpg)

> **注意：** 根据自己需求选择。

---

![](media/14797132371800/14804090316285.jpg)

---

![](media/14797132371800/14804090549203.jpg)

---

![](media/14797132371800/14799701685609.jpg)

> **注意：** 在上文中的 `初始化 Activity Monitor, Hive Metastore Server, Oozie Server 数据库` 章节，我们已经在 `外部` 数据库（ `mysql` ）中为这三个服务创建好了数据库。分别对应关系是：`Hive` -> `metastore` , `Activity Monitor` -> `amon` , `Oozie Server` -> `oozie` 。在其设置页面中直接选择 `Mysql` 数据库并且填入对应数据库名称即可。

![](media/14797132371800/14915520840098.jpg)

> **注意：** 如果需要配置多目录，请按照上图设置。

![](media/14797132371800/14999338546170.jpg)

等待安装完成

![](media/14797132371800/14999338879351.jpg)

## 测试安装结果

### 安装成功页面

![](media/14797132371800/14804053856456.jpg)

### 执行测试命令 `计算 PI`

```bash
# 执行测试命令，计算 `pi` 
> sudo -u hdfs hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100

Number of Maps  = 10
Samples per Map = 100
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Starting Job
16/11/29 15:38:56 INFO client.RMProxy: Connecting to ResourceManager at server349/10.0.3.49:8032
16/11/29 15:38:57 INFO input.FileInputFormat: Total input paths to process : 10
16/11/29 15:38:57 INFO mapreduce.JobSubmitter: number of splits:10
16/11/29 15:38:57 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1479971110467_0002
16/11/29 15:38:57 INFO impl.YarnClientImpl: Submitted application application_1479971110467_0002
16/11/29 15:38:57 INFO mapreduce.Job: The url to track the job: http://server349:8088/proxy/application_1479971110467_0002/
16/11/29 15:38:57 INFO mapreduce.Job: Running job: job_1479971110467_0002
16/11/29 15:39:02 INFO mapreduce.Job: Job job_1479971110467_0002 running in uber mode : false
16/11/29 15:39:02 INFO mapreduce.Job:  map 0% reduce 0%
16/11/29 15:39:08 INFO mapreduce.Job:  map 100% reduce 0%
16/11/29 15:39:14 INFO mapreduce.Job:  map 100% reduce 100%
16/11/29 15:39:14 INFO mapreduce.Job: Job job_1479971110467_0002 completed successfully
16/11/29 15:39:14 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=98
                FILE: Number of bytes written=1272159
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=2630
                HDFS: Number of bytes written=215
                HDFS: Number of read operations=43
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=3
        Job Counters 
                Launched map tasks=10
                Launched reduce tasks=1
                Data-local map tasks=10
                Total time spent by all maps in occupied slots (ms)=27842
                Total time spent by all reduces in occupied slots (ms)=2706
                Total time spent by all map tasks (ms)=27842
                Total time spent by all reduce tasks (ms)=2706
                Total vcore-seconds taken by all map tasks=27842
                Total vcore-seconds taken by all reduce tasks=2706
                Total megabyte-seconds taken by all map tasks=28510208
                Total megabyte-seconds taken by all reduce tasks=2770944
        Map-Reduce Framework
                Map input records=10
                Map output records=20
                Map output bytes=180
                Map output materialized bytes=340
                Input split bytes=1450
                Combine input records=0
                Combine output records=0
                Reduce input groups=2
                Reduce shuffle bytes=340
                Reduce input records=20
                Reduce output records=0
                Spilled Records=40
                Shuffled Maps =10
                Failed Shuffles=0
                Merged Map outputs=10
                GC time elapsed (ms)=224
                CPU time spent (ms)=7050
                Physical memory (bytes) snapshot=6275936256
                Virtual memory (bytes) snapshot=18447077376
                Total committed heap usage (bytes)=9065988096
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters 
                Bytes Read=1180
        File Output Format Counters 
                Bytes Written=97
Job Finished in 17.364 seconds
Estimated value of Pi is 3.14800000000000000000
```

## 卸载 `Cloudera Manager`

> 参考官网文档：(`https://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_ig_uninstall_cm.html`)

```bash
# 删除组件目录
> rm -rf /var/lib/flume-ng /var/lib/hadoop* /var/lib/hue /var/lib/navigator /var/lib/oozie /var/lib/solr /var/lib/sqoop* /var/lib/zookeeper

# 删除组件数据目录
> rm -rf /dfs /yarn /mapred

# 进入页面停止所有服务，包括`集群服务`和`监控服务`
# 进入页面删除`集群`和`监控`

> sudo service cloudera-scm-server stop
Stopping cloudera-scm-server:                              [  OK  ]

# 可能会没有这个服务
> sudo service cloudera-scm-server-db stop
cloudera-scm-server-db: unrecognized service

> sudo yum remove cloudera-manager-server
[...]

# 可能会没有这个服务
> sudo yum remove cloudera-manager-server-db-2
[...]

# 所有服务器执行
> sudo service cloudera-scm-agent stop
Stopping cloudera-scm-agent:                               [  OK  ]

# 所有服务器执行
> sudo yum remove 'cloudera-manager-*'
[...]

# 所有服务器执行
> sudo yum remove 'cloudera-manager-*' avro-tools crunch flume-ng hadoop-hdfs-fuse hadoop-hdfs-nfs3 hadoop-httpfs hadoop-kms hbase-solr hive-hbase hive-webhcat hue-beeswax hue-hbase hue-impala hue-pig hue-plugins hue-rdbms hue-search hue-spark hue-sqoop hue-zookeeper impala impala-shell kite llama mahout oozie pig pig-udf-datafu search sentry solr-mapreduce spark-core spark-master spark-worker spark-history-server spark-python sqoop sqoop2 whirr hue-common oozie-client solr solr-doc sqoop2-client zookeeper
[...]

# 所有服务器执行，杀进程
> for u in cloudera-scm flume hadoop hdfs hbase hive httpfs hue impala llama mapred oozie solr spark sqoop sqoop2 yarn zookeeper; do sudo kill $(ps -u $u -o pid=); done

# 删除`Cloudera Manager`数据
# 所哟服务器执行
> sudo umount cm_processes 
> sudo rm -Rf /usr/share/cmf /var/lib/cloudera* /var/cache/yum/cloudera* /var/log/cloudera* /var/run/cloudera*
> sudo rm /tmp/.scm_prepare_node.lock
> sudo rm -Rf /var/lib/flume-ng /var/lib/hadoop* /var/lib/hue /var/lib/navigator /var/lib/oozie /var/lib/solr /var/lib/sqoop* /var/lib/zookeeper

# 删除外部数据库
> mysql -uroot -proot123
[...]

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| ... 				    |
| amon               |
| metastore          |
| ...                |
| oozie              |
| ...                |
| scm                |
+--------------------+
7 rows in set (0.00 sec)

mysql> drop database amon;    
Query OK, 61 rows affected (2.00 sec)

mysql> drop database metastore;
Query OK, 53 rows affected (1.00 sec)

mysql> drop database oozie;    
Query OK, 12 rows affected (0.84 sec)

mysql> drop database scm;  
Query OK, 42 rows affected (1.13 sec)

mysql> exit;
Bye
```

## `Cloudera Manager` 客户端部署

> **方法：**首先将需要部署客户端的服务器添加到`Cloudera Manager`集群中；其次设置该服务器为`Hbase`, `HDFS`, `Hive`, `YARN`的`Gateway`角色；最后使用集群提供的`部署客户端配置`完成客户端配置文件的分发。

### 添加服务器到`Cloudera Manager`集群中

> **特别注意：执行添加操作前，需要保证该服务器的`信任关系`、`SELinux`、`防火墙`、`必要软件`等已经安装和配置完成，具体配置步骤详见上文安装文档。**

![](media/14797132371800/14930259041023.jpg)

![](media/14797132371800/14930259216778.jpg)

![](media/14797132371800/14930259407929.jpg)

> **特别注意：该步骤之前保证所有服务器的`/etc/hosts`文件中有需要添加服务器的主机名。**

![](media/14797132371800/14930260194878.jpg)

> **特别注意：`自定义存储库` 和 `自定义GPG` 需要和上文中安装步骤的内容一致。同时要保证该服务器的`/etc/hosts`文件中有配置 `archive.centrin.com.cn` 对应的 `IP地址`。**

![](media/14797132371800/14930262161091.jpg)

![](media/14797132371800/14930263843212.jpg)

![](media/14797132371800/14930264055253.jpg)

![](media/14797132371800/14930265963843.jpg)

![](media/14797132371800/14930267048709.jpg)

![](media/14797132371800/14930267375734.jpg)

> **特别注意：红框中问题解决方法详见上文安装步骤。**
> 解决问题后可以点击页面最上端的`重新运行`按钮重新检查。
> 
> `检查 /etc/hosts 时发现以下错误...` 和 `检查主机名称时发现以下失败结果。仅显示前 1000 个失败结果...` 不需要解决

![](media/14797132371800/14930270147294.jpg)

![](media/14797132371800/14930270338679.jpg)

![](media/14797132371800/14930270546246.jpg)


![](media/14797132371800/14930270683686.jpg)

### 设置角色

#### `Hbase`

![](media/14797132371800/14930271423850.jpg)

![](media/14797132371800/14930271565056.jpg)

![](media/14797132371800/14930271706322.jpg)

![](media/14797132371800/14930271844122.jpg)

![](media/14797132371800/14930272038106.jpg)

![](media/14797132371800/14930272196057.jpg)

![](media/14797132371800/14930272314066.jpg)

#### `HDFS`

![](media/14797132371800/14930272605772.jpg)

![](media/14797132371800/14930272727175.jpg)

![](media/14797132371800/14930272848776.jpg)

![](media/14797132371800/14930273047351.jpg)

![](media/14797132371800/14930273208193.jpg)

![](media/14797132371800/14930273459553.jpg)

![](media/14797132371800/14930273610639.jpg)

#### `Hive` 

![](media/14797132371800/14930273919895.jpg)

![](media/14797132371800/14930274047699.jpg)

![](media/14797132371800/14930274194385.jpg)

![](media/14797132371800/14930274360248.jpg)

![](media/14797132371800/14930274533155.jpg)

![](media/14797132371800/14930274669285.jpg)

![](media/14797132371800/14930274929441.jpg)

#### `YARN`

![](media/14797132371800/14930275210966.jpg)

![](media/14797132371800/14930275349437.jpg)

![](media/14797132371800/14930275458127.jpg)

![](media/14797132371800/14930275560290.jpg)

![](media/14797132371800/14930275788449.jpg)

![](media/14797132371800/14930275957980.jpg)

![](media/14797132371800/14930276166105.jpg)

#### `部署客户端配置`

![](media/14797132371800/14930276572788.jpg)

![](media/14797132371800/14930276743018.jpg)

![](media/14797132371800/14930276988803.jpg)

![](media/14797132371800/14930277291354.jpg)

![](media/14797132371800/14930277377297.jpg)

#### 检查

```bash
# 登陆到 `server350` 服务器
# 检查`HDFS`
> hdfs dfs -ls /
Found 4 items
drwxrwxrwx   - ccicall ccicall             0 2017-04-24 11:12 /gfbak
drwxr-xr-x   - hbase   hbase               0 2017-04-05 16:10 /hbase
drwxrwxrwt   - hdfs    supergroup          0 2017-04-24 16:17 /tmp
drwxrwxrwx   - hdfs    supergroup          0 2017-04-01 18:02 /user

# 检查`Hbase`
> hbase shell
[...]
hbase> list
TABLE                                                                                                             
asdf                                                                                                              
regist_businessInfo                                                                                               
scoreQuality                                                                                                      
scoreQualityGroup                                                                                                 
smartv                                                                                                            
smartv_black_list                                                                                                 
smartv_new                                                                                                        
smartv_property                                                                                                   
smartv_test                                                                                                       
smartv_voiceprint_temp                                                                                            
test                                                                                                              
waveform                                                                                                          
12 row(s) in 0.1900 seconds

=> ["asdf", "regist_businessInfo", "scoreQuality", "scoreQualityGroup", "smartv", "smartv_black_list", "smartv_new", "smartv_property", "smartv_test", "smartv_voiceprint_temp", "test", "waveform"]
hbase> exit

# 检查`Hive`
> hive
[...]
hive> show tables;
OK
smartv_hive_main
smartv_quality_detail
Time taken: 1.611 seconds, Fetched: 2 row(s)
hive> exit;
```

`-EOF-`


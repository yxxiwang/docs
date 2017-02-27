## `ANT`安装文档

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

##### 安装方法

###### 一般组件安装（使用`yum`）

```bash
# 切换成为`root`用户
> su - 
Password: 

> yum -y install openssh-clients vim
[...]
```

### 安装流程

#### 获取组件

```bash
# 切换用户
> su - ccicall
[...]

# 切换目录，若没有目录请切换`root`用户进行创建，并且付给`ccicall`用户权限
> cd /ccicall/opt
> pwd
/ccicall/opt

# 获取组件
> scp root@10.0.3.45:/Archive/Project_Dadi/02.sources/apache-ant-1.9.7-bin.tar.gz ./
[...]

# 验证
> ll apache-ant-1.9.7-bin.tar.gz 
-rw-r--r-- 1 ccicall ccicall 5601575 Feb 14 15:03 apache-ant-1.9.7-bin.tar.gz

# 解压
> tar -zxvf apache-ant-1.9.7-bin.tar.gz
> ll apache-ant-1.9.7
total 432
drwxr-xr-x 2 ccicall ccicall   4096 Feb 14 15:04 bin
-rw-r--r-- 1 ccicall ccicall   6081 Apr  9  2016 CONTRIBUTORS
-rw-r--r-- 1 ccicall ccicall  30018 Apr  9  2016 contributors.xml
drwxr-xr-x 3 ccicall ccicall   4096 Feb 14 15:04 etc
-rw-r--r-- 1 ccicall ccicall  11253 Apr  9  2016 fetch.xml
-rw-r--r-- 1 ccicall ccicall   4445 Apr  9  2016 get-m2.xml
-rw-r--r-- 1 ccicall ccicall    126 Apr  9  2016 INSTALL
-rw-r--r-- 1 ccicall ccicall  92261 Apr  9  2016 KEYS
drwxr-xr-x 2 ccicall ccicall   4096 Feb 14 15:04 lib
-rw-r--r-- 1 ccicall ccicall  15289 Apr  9  2016 LICENSE
drwxr-xr-x 8 ccicall ccicall   4096 Feb 14 15:04 manual
-rw-r--r-- 1 ccicall ccicall    305 Apr  9  2016 NOTICE
-rw-r--r-- 1 ccicall ccicall   1915 Apr  9  2016 patch.xml
-rw-r--r-- 1 ccicall ccicall   4119 Apr  9  2016 README
-rw-r--r-- 1 ccicall ccicall 233424 Apr  9  2016 WHATSNEW

# 将`ant`添加至环境变量中（`~/.bash_profile`）
> vim ~/.bash_profile
```

> 因为这里之前安装了`JAVA`，如果没安装`JAVA`请先安装，安装方法请详见`JAVA`安装文档。

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

export JAVA_HOME=/ccicall/opt/jdk1.7.0_76
export ANT_HOME=/ccicall/opt/apache-ant-1.9.7
export PATH=${JAVA_HOME}/bin:${ANT_HOME}/bin:${PATH}
export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
```

```bash
# 验证
> su - ccicall
[...]

> ant -version
Apache Ant(TM) version 1.9.7 compiled on April 9 2016

# 删除组件包
> ll /ccicall/opt/apache-ant-1.9.7-bin.tar.gz 
-rw-r--r-- 1 ccicall ccicall 5601575 Feb 14 15:03 /ccicall/opt/apache-ant-1.9.7-bin.tar.gz

> rm -rf /ccicall/opt/apache-ant-1.9.7-bin.tar.gz

> ll /ccicall/opt/apache-ant-1.9.7-bin.tar.gz 
ls: cannot access /ccicall/opt/apache-ant-1.9.7-bin.tar.gz: No such file or directory
```

`-EOF-`



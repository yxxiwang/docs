## `JAVA`安装文档

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

```bash
# 切换用户
> su - 

# 创建目录
> mkdir -p /ccicall/opt/

# 赋权
> chown -R ccicall:ccicall /ccicall

# 检查
> ll -d /ccicall/
drwxr-xr-x. 2 ccicall ccicall 4096 Feb  9 15:57 /ccicall/

# 切换用户
> su - ccicall

# 获取组件
> cd /ccicall/opt/
> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/jdk-7u76-linux-x64.tar.gz ./

> ll
total 138916
-rw-r--r--. 1 ccicall ccicall 142249690 Feb  9 16:01 jdk-7u76-linux-x64.tar.gz

# 解压
> tar -zxvf jdk-7u76-linux-x64.tar.gz

> ll jdk1.7.0_76/
total 19772
drwxr-xr-x. 2 ccicall ccicall     4096 Dec 19  2014 bin
-r--r--r--. 1 ccicall ccicall     3339 Dec 19  2014 COPYRIGHT
drwxr-xr-x. 4 ccicall ccicall     4096 Dec 19  2014 db
drwxr-xr-x. 3 ccicall ccicall     4096 Dec 19  2014 include
drwxr-xr-x. 5 ccicall ccicall     4096 Dec 19  2014 jre
drwxr-xr-x. 5 ccicall ccicall     4096 Dec 19  2014 lib
-r--r--r--. 1 ccicall ccicall       40 Dec 19  2014 LICENSE
drwxr-xr-x. 4 ccicall ccicall     4096 Dec 19  2014 man
-r--r--r--. 1 ccicall ccicall      114 Dec 19  2014 README.html
-rw-r--r--. 1 ccicall ccicall      499 Dec 19  2014 release
-rw-r--r--. 1 ccicall ccicall 19917352 Dec 19  2014 src.zip
-rw-r--r--. 1 ccicall ccicall   110114 Dec 18  2014 THIRDPARTYLICENSEREADME-JAVAFX.txt
-r--r--r--. 1 ccicall ccicall   173559 Dec 19  2014 THIRDPARTYLICENSEREADME.txt

# 备份
> cp ~/.bash_profile ~/.bash_profile.raw

> ll -a ~/.bash_profile*
-rw-r--r--. 1 ccicall ccicall 176 Jul  9  2013 /home/ccicall/.bash_profile
-rw-r--r--. 1 ccicall ccicall 176 Feb  9 16:05 /home/ccicall/.bash_profile.raw

# 修改内容
> vim ~/.bash_profile
```

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
export PATH=${JAVA_HOME}/bin:${PATH}
export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
```

```bash
# 对比修改后文件
> git diff ~/.bash_profile.raw ~/.bash_profile
diff --git a/home/ccicall/.bash_profile.raw b/home/ccicall/.bash_profile
index 3dc099a..f0909e6 100644
--- a/home/ccicall/.bash_profile.raw
+++ b/home/ccicall/.bash_profile
@@ -10,3 +10,7 @@ fi
 PATH=$PATH:$HOME/bin
 
 export PATH
+
+export JAVA_HOME=/ccicall/opt/jdk1.7.0_76
+export PATH=${JAVA_HOME}/bin:${PATH}
+export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar

# 验证
> su - ccicall
> java -version
java version "1.7.0_76"
Java(TM) SE Runtime Environment (build 1.7.0_76-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.76-b04, mixed mode)

# 删除组件包
> rm -rf /ccicall/opt/jdk-7u76-linux-x64.tar.gz

# 验证
> ll /ccicall/opt/
total 4
drwxr-xr-x. 8 ccicall ccicall 4096 Dec 19  2014 jdk1.7.0_76

# 在`/usr/bin/`目录中创建软链接
#!# 有的时候如果不在`/usr/bin/`目录中创建软链接的话，通过`ssh root@ip 'java -version'`会包找不到java的错误
> su - 
> ln -s /ccicall/opt/jdk1.7.0_76/bin/java /usr/bin/java
> ll /usr/bin/java
lrwxrwxrwx 1 root root 34 Mar  9 17:00 /usr/bin/java -> /ccicall/opt/jdk1.7.0_76/bin/java
```

`-EOF-`



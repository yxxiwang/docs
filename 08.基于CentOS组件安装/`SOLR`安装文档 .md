## `SOLR`安装文档 

### 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

### 目录

[TOC]

### 部署环境说明

名称|版本|缩写|默认安装软件组
---|---|---|---
`RedHat-Enterprise-Linux`|`6.5`|`RHEL-6.5`|`Minimal`

#### 服务器信息

角色|`IP`地址|主机名（`HOSTNAME`）|操作系统|配置
---|---|---|---|---
`主节点服务器（Solr Master Server）`|`10.0.3.48`|`server348`|`RHEL-6.5`|`10c,64G`
`从节点服务器（Solr Slave Server）`|`10.0.3.50`|`server350`|`RHEL-6.5`|`10c,64G`
`软件服务器`|`10.0.3.45`|`server345`|`RHEL-6.5`|`10c,64G`

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

#### 安装`主节点服务器（Solr Master Server）`

```bash
# 登陆`主节点服务器（Solr Master Server）`
> /sbin/ifconfig 
eth1      Link encap:Ethernet  HWaddr A0:36:9F:8C:2A:EB  
          inet addr:10.0.3.48  Bcast:10.0.3.255  Mask:255.255.255.0
          inet6 addr: fe80::a236:9fff:fe8c:2aeb/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4586107 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1007034 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:6489296980 (6.0 GiB)  TX bytes:600257282 (572.4 MiB)
          Memory:c7400000-c7500000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:216740 errors:0 dropped:0 overruns:0 frame:0
          TX packets:216740 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:19409600 (18.5 MiB)  TX bytes:19409600 (18.5 MiB)

# 切换用户
> su - ccicall

# 切换目录
> cd /ccicall/opt/
> pwd
/ccicall/opt

# 获取软件包
> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr安装/solr-5.5.1-master.zip ./
[...]

# 检查
> ll solr-5.5.1-master.zip 
-rw-r--r-- 1 ccicall ccicall 188833867 Feb 16 10:14 solr-5.5.1-master.zip

# 解压
> unzip solr-5.5.1-master.zip
[...]

# 检查
> ll solr-5.5.1
total 1240
drwxr-xr-x  3 ccicall ccicall   4096 Aug  9  2016 bin
-rw-r--r--  1 ccicall ccicall 555321 May  1  2016 CHANGES.txt
drwxr-xr-x 13 ccicall ccicall   4096 May  1  2016 contrib
drwxr-xr-x  4 ccicall ccicall   4096 Jun 12  2016 dist
drwxr-xr-x 19 ccicall ccicall   4096 Jun 12  2016 docs
drwxr-xr-x  8 ccicall ccicall   4096 Jun 12  2016 example
drwxr-xr-x  2 ccicall ccicall  36864 Jun 12  2016 licenses
-rw-r--r--  1 ccicall ccicall  12646 Feb  1  2016 LICENSE.txt
-rw-r--r--  1 ccicall ccicall 590277 May  1  2016 LUCENE_CHANGES.txt
-rw-r--r--  1 ccicall ccicall  26529 Feb  1  2016 NOTICE.txt
-rw-r--r--  1 ccicall ccicall   7162 May  1  2016 README.txt
drwxr-xr-x 11 ccicall ccicall   4096 Jun 12  2016 server
drwxr-xr-x 11 ccicall ccicall   4096 Jun 15  2016 smartv

# 切换到`solr`目录
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

# 删除原有`pid`文件
> rm -rf bin/*.pid

# 删除原有`zookeeper`文件
> rm -rf smartv/solr/zoo_data/*
 
# 删除原有日志文件
> rm -rf smartv/logs/*

# 创建索引存储文件目录
# 切换用户
> su - 
> mkdir -p /data/solr/collection1/
> chmod -R 777 /data/solr/
> chown -R ccicall:ccicall /data/solr/
> su - ccicall

# 检查
> ll -d /data/solr/
drwxrwxrwx 3 ccicall ccicall 4096 Feb 16 10:24 /data/solr/

# 备份配置文件
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd 
/ccicall/opt/solr-5.5.1

# 备份配置文件`solr.in.sh`
> cp bin/solr.in.sh  bin/solr.in.sh.raw
> ll bin/solr.in.sh*
-rwxr-xr-x 1 ccicall ccicall 5194 Jun 16  2016 bin/solr.in.sh
-rwxr-xr-x 1 ccicall ccicall 5194 Feb 16 12:01 bin/solr.in.sh.raw

# 修改配置文件`solr.in.sh`
# 修改第62行，`BOOTSTARP_CONF_DIR`，
# 将 /opt/solr-5.5.1/smartv/solr/collection1/conf 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf
# 原始：
# # SolrCloud模式启动Collection配置
# # 配置文件目录
# BOOTSTARP_CONF_DIR="/opt/solr-5.5.1/smartv/solr/collection1/conf"
# 
# 修改后：
# # SolrCloud模式启动Collection配置
# # 配置文件目录
# BOOTSTARP_CONF_DIR="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf"
> vim bin/solr.in.sh

# 对比修改后文件内容
> git diff bin/solr.in.sh.raw bin/solr.in.sh
diff --git a/bin/solr.in.sh.raw b/bin/solr.in.sh
index b6adcb9..cbcf8a6 100755
--- a/bin/solr.in.sh.raw
+++ b/bin/solr.in.sh
@@ -59,7 +59,7 @@ ZK_CLIENT_TIMEOUT="600000"
 
 # SolrCloud模式启动Collection配置
 # 配置文件目录
-BOOTSTARP_CONF_DIR="/opt/solr-5.5.1/smartv/solr/collection1/conf"
+BOOTSTARP_CONF_DIR="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf"

# 备份配置文件`solrconfig.xml`
> cp smartv/solr/collection1/conf/solrconfig.xml smartv/solr/collection1/conf/solrconfig.xml.raw
> ll smartv/solr/collection1/conf/solrconfig.xml*
-rw-r--r-- 1 ccicall ccicall 71835 Jun 15  2016 smartv/solr/collection1/conf/solrconfig.xml
-rw-r--r-- 1 ccicall ccicall 71835 Feb 16 11:17 smartv/solr/collection1/conf/solrconfig.xml.raw

# 修改配置文件`solrconfig.xml`
# 修改第800行，`dicPath`，将
# /opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/ 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/
#
# 原始：
# <!-- mmseg4j -->
# <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
#   <lst name="defaults">
#     <str name="dicPath">/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
#     <str name="reload">true</str>
#   </lst>
# </requestHandler>
#
# 修改后：
# <!-- mmseg4j -->
# <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
#   <lst name="defaults">
#     <str name="dicPath">/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
#     <str name="reload">true</str>
#   </lst>
# </requestHandler>
> vim smartv/solr/collection1/conf/solrconfig.xml

# 对比修改后内容 
> git diff smartv/solr/collection1/conf/solrconfig.xml.raw smartv/solr/collection1/conf/solrconfig.xml
diff --git a/smartv/solr/collection1/conf/solrconfig.xml.raw b/smartv/solr/collection1/conf/solrconfig.xml
index 225561a..06df1a4 100644
--- a/smartv/solr/collection1/conf/solrconfig.xml.raw
+++ b/smartv/solr/collection1/conf/solrconfig.xml
@@ -797,7 +797,7 @@
   <!-- mmseg4j -->
   <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
     <lst name="defaults">
-      <str name="dicPath">/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
+      <str name="dicPath">/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
       <str name="reload">true</str>
     </lst>
   </requestHandler>

# 备份配置文件`schema.xml`
> cp smartv/solr/collection1/conf/schema.xml smartv/solr/collection1/conf/schema.xml.raw
> ll smartv/solr/collection1/conf/schema.xml*
-rw-r--r-- 1 ccicall ccicall 68803 Jun 16  2016 smartv/solr/collection1/conf/schema.xml
-rw-r--r-- 1 ccicall ccicall 68803 Feb 16 11:23 smartv/solr/collection1/conf/schema.xml.raw

# 修改配置文件`schema.xml`
# 分别修改第424, 430, 439行，`dicPath`，将
# /opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/ 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/
# 
# 原始：
# <!-- mmseg4j-solr -->
# <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>
#   </analyzer>
# </fieldtype>
#
# <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
#   <analyzer type="index">
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
#   </analyzer>
#   <analyzer type="query">
#   <tokenizer class="solr.WhitespaceTokenizerFactory"/>
#     </analyzer>
# </fieldtype>
#
# <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
# </fieldtype>
# 
# 修改后：
# <!-- mmseg4j-solr -->
# <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>
#   </analyzer>
# </fieldtype>
#
# <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
#   <analyzer type="index">
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
#   <analyzer type="query">
#   <tokenizer class="solr.WhitespaceTokenizerFactory"/>
#     </analyzer>
# </fieldtype>
#
# <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
# </fieldtype>
> vim smartv/solr/collection1/conf/schema.xml

# 对比修改后文件
> git diff smartv/solr/collection1/conf/schema.xml.raw smartv/solr/collection1/conf/schema.xml
diff --git a/smartv/solr/collection1/conf/schema.xml.raw b/smartv/solr/collection1/conf/schema.xml
index ffdc594..a39ef8f 100644
--- a/smartv/solr/collection1/conf/schema.xml.raw
+++ b/smartv/solr/collection1/conf/schema.xml
@@ -421,13 +421,13 @@
     <!-- mmseg4j-solr -->
     <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
       <analyzer>
-         <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>           
+         <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>      
       </analyzer>
     </fieldtype>
 
     <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
       <analyzer type="index">
-        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
+        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
       </analyzer>
       <analyzer type="query">
         <tokenizer class="solr.WhitespaceTokenizerFactory"/>
@@ -436,7 +436,7 @@
 
     <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
       <analyzer>
-        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
+        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
       </analyzer>
     </fieldtype>

# 需要安装`lsof`组件
> su -
> yum -y install lsof
Loaded plugins: product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package lsof.x86_64 0:4.82-4.el6 will be installed
--> Finished Dependency Resolution
[...]
Installed:
  lsof.x86_64 0:4.82-4.el6                                                                                 

Complete!

# 为了解决`solr5.x`版本默认不能使用`iframe`的问题，需要替换核心包
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

# 替换核心包
> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar dist/
> md5sum dist/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  dist/solr-core-5.5.1.jar

> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar server/solr-webapp/webapp/WEB-INF/lib/
> md5sum server/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  server/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar

> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar smartv/solr-webapp/webapp/WEB-INF/lib/
> md5sum smartv/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  smartv/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar

# 启动服务
#!# 启动之前要为用户安装`JAVA`，若为安装请详见`JAVA`安装文档
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

> bin/solr restart -c -m 8g -d smartv/
Waiting up to 30 seconds to see Solr running on port 8983 [/]  
Started Solr server on port 8983 (pid=6681). Happy searching!
```

> 进入页面：`http://10.0.3.48:8983/` 查看启动情况

![](media/14872110765844/14872183020261.jpg)


> 进入页面 `http://10.0.3.48:8983/solr/#/~cloud` 查看集群情况

![](media/14872110765844/14872183535632.jpg)

#### 安装`从节点服务器（Solr Slave Server）`

```bash
# 登陆`从节点服务器（Solr Slave Server）`
> /sbin/ifconfig 
eth1      Link encap:Ethernet  HWaddr A0:36:9F:8C:2C:43  
          inet addr:10.0.3.50  Bcast:10.0.3.255  Mask:255.255.255.0
          inet6 addr: fe80::a236:9fff:fe8c:2c43/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1564119 errors:0 dropped:0 overruns:0 frame:0
          TX packets:364469 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1975689163 (1.8 GiB)  TX bytes:27532688 (26.2 MiB)
          Memory:c7400000-c7500000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:4431 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4431 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:818204 (799.0 KiB)  TX bytes:818204 (799.0 KiB)

# 切换用户
> su - ccicall

# 切换目录
> cd /ccicall/opt/
> pwd
/ccicall/opt

# 获取软件包
> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr安装/solr-5.5.1-slave.zip ./
[...]

# 检查
> ll solr-5.5.1-slave.zip
-rw-r--r-- 1 ccicall ccicall 185768542 Feb 16 13:14 solr-5.5.1-slave.zip

# 解压
> unzip solr-5.5.1-slave.zip
[...]

# 检查
> ll solr-5.5.1
total 1240
drwxr-xr-x  3 ccicall ccicall   4096 Aug  9  2016 bin
-rw-r--r--  1 ccicall ccicall 555321 May  1  2016 CHANGES.txt
drwxr-xr-x 13 ccicall ccicall   4096 May  1  2016 contrib
drwxr-xr-x  4 ccicall ccicall   4096 Jun 12  2016 dist
drwxr-xr-x 19 ccicall ccicall   4096 Jun 12  2016 docs
drwxr-xr-x  7 ccicall ccicall   4096 Jun 12  2016 example
drwxr-xr-x  2 ccicall ccicall  36864 Jun 12  2016 licenses
-rw-r--r--  1 ccicall ccicall  12646 Feb  1  2016 LICENSE.txt
-rw-r--r--  1 ccicall ccicall 590277 May  1  2016 LUCENE_CHANGES.txt
-rw-r--r--  1 ccicall ccicall  26529 Feb  1  2016 NOTICE.txt
-rw-r--r--  1 ccicall ccicall   7162 May  1  2016 README.txt
drwxr-xr-x 11 ccicall ccicall   4096 Jun 12  2016 server
drwxr-xr-x 11 ccicall ccicall   4096 Jun 16  2016 smart

# 切换到`solr`目录
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

# 删除原有`pid`文件
> rm -rf bin/*.pid

# 删除原有`zookeeper`文件
> rm -rf smartv/solr/zoo_data/*
 
# 删除原有日志文件
> rm -rf smartv/logs/*

# 创建索引存储文件目录
# 切换用户
> su - 
> mkdir -p /data/solr/collection1/
> chmod -R 777 /data/solr/
> chown -R ccicall:ccicall /data/solr/
> su - ccicall

# 检查
> ll -d /data/solr/
drwxrwxrwx 3 ccicall ccicall 4096 Feb 16 10:24 /data/solr/

# 备份配置文件
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd 
/ccicall/opt/solr-5.5.1

# 备份配置文件`solr.in.sh`
> cp bin/solr.in.sh  bin/solr.in.sh.raw
> ll bin/solr.in.sh*
-rwxr-xr-x 1 ccicall ccicall 5194 Jun 16  2016 bin/solr.in.sh
-rwxr-xr-x 1 ccicall ccicall 5194 Feb 16 12:01 bin/solr.in.sh.raw

# 修改配置文件`solr.in.sh`
# 修改第51行，`ZK_HOST`，修改为`主节点服务器（Solr Master Server）`的地址
# 原始：
# ZK_HOST="10.0.1.15:9983"
# 修改后：
# ZK_HOST="10.0.3.48:9983"
#
# 修改第62行，`BOOTSTARP_CONF_DIR`，
# 将 /opt/solr-5.5.1/smartv/solr/collection1/conf 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf
# 原始：
# # SolrCloud模式启动Collection配置
# # 配置文件目录
# BOOTSTARP_CONF_DIR="/opt/solr-5.5.1/smartv/solr/collection1/conf"
# 
# 修改后：
# # SolrCloud模式启动Collection配置
# # 配置文件目录
# BOOTSTARP_CONF_DIR="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf"
> vim bin/solr.in.sh

# 对比修改后文件内容
> git diff bin/solr.in.sh.raw bin/solr.in.sh
diff --git a/bin/solr.in.sh.raw b/bin/solr.in.sh
index fe1085d..b7209a8 100755
--- a/bin/solr.in.sh.raw
+++ b/bin/solr.in.sh
@@ -48,7 +48,7 @@ GC_TUNE=-XX:NewRatio=3 \
 # Set the ZooKeeper connection string if using an external ZooKeeper ensemble
 # e.g. host1:2181,host2:2181/chroot
 # Leave empty if not using SolrCloud
-ZK_HOST="10.0.1.15:9983"
+ZK_HOST="10.0.3.48:9983"
 
 # Set the ZooKeeper client timeout (for SolrCloud mode)
 ZK_CLIENT_TIMEOUT="600000"
@@ -59,7 +59,7 @@ ZK_CLIENT_TIMEOUT="600000"
 
 # SolrCloud模式启动Collection配置
 # 配置文件目录
-BOOTSTARP_CONF_DIR="/opt/solr-5.5.1/smartv/solr/collection1/conf"
+BOOTSTARP_CONF_DIR="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf"

# 备份配置文件`solrconfig.xml`
> cp smartv/solr/collection1/conf/solrconfig.xml smartv/solr/collection1/conf/solrconfig.xml.raw
> ll smartv/solr/collection1/conf/solrconfig.xml*
-rw-r--r-- 1 ccicall ccicall 71835 Jun 15  2016 smartv/solr/collection1/conf/solrconfig.xml
-rw-r--r-- 1 ccicall ccicall 71835 Feb 16 11:17 smartv/solr/collection1/conf/solrconfig.xml.raw

# 修改配置文件`solrconfig.xml`
# 修改第800行，`dicPath`，将
# /opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/ 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/
#
# 原始：
# <!-- mmseg4j -->
# <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
#   <lst name="defaults">
#     <str name="dicPath">/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
#     <str name="reload">true</str>
#   </lst>
# </requestHandler>
#
# 修改后：
# <!-- mmseg4j -->
# <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
#   <lst name="defaults">
#     <str name="dicPath">/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
#     <str name="reload">true</str>
#   </lst>
# </requestHandler>
> vim smartv/solr/collection1/conf/solrconfig.xml

# 对比修改后内容 
> git diff smartv/solr/collection1/conf/solrconfig.xml.raw smartv/solr/collection1/conf/solrconfig.xml
diff --git a/smartv/solr/collection1/conf/solrconfig.xml.raw b/smartv/solr/collection1/conf/solrconfig.xml
index 225561a..06df1a4 100644
--- a/smartv/solr/collection1/conf/solrconfig.xml.raw
+++ b/smartv/solr/collection1/conf/solrconfig.xml
@@ -797,7 +797,7 @@
   <!-- mmseg4j -->
   <requestHandler name="/mmseg4j" class="com.chenlb.mmseg4j.solr.MMseg4jHandler" >
     <lst name="defaults">
-      <str name="dicPath">/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
+      <str name="dicPath">/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/</str>
       <str name="reload">true</str>
     </lst>
   </requestHandler>

# 备份配置文件`schema.xml`
> cp smartv/solr/collection1/conf/schema.xml smartv/solr/collection1/conf/schema.xml.raw
> ll smartv/solr/collection1/conf/schema.xml*
-rw-r--r-- 1 ccicall ccicall 68803 Jun 16  2016 smartv/solr/collection1/conf/schema.xml
-rw-r--r-- 1 ccicall ccicall 68803 Feb 16 11:23 smartv/solr/collection1/conf/schema.xml.raw

# 修改配置文件`schema.xml`
# 分别修改第424, 430, 439行，`dicPath`，将
# /opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/ 修改为 /ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/
# 
# 原始：
# <!-- mmseg4j-solr -->
# <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>
#   </analyzer>
# </fieldtype>
#
# <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
#   <analyzer type="index">
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
#   </analyzer>
#   <analyzer type="query">
#   <tokenizer class="solr.WhitespaceTokenizerFactory"/>
#     </analyzer>
# </fieldtype>
#
# <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
# </fieldtype>
# 
# 修改后：
# <!-- mmseg4j-solr -->
# <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>
#   </analyzer>
# </fieldtype>
#
# <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
#   <analyzer type="index">
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
#   <analyzer type="query">
#   <tokenizer class="solr.WhitespaceTokenizerFactory"/>
#     </analyzer>
# </fieldtype>
#
# <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
#   <analyzer>
#     <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
#   </analyzer>
# </fieldtype>
> vim smartv/solr/collection1/conf/schema.xml

# 对比修改后文件
> git diff smartv/solr/collection1/conf/schema.xml.raw smartv/solr/collection1/conf/schema.xml
diff --git a/smartv/solr/collection1/conf/schema.xml.raw b/smartv/solr/collection1/conf/schema.xml
index ffdc594..a39ef8f 100644
--- a/smartv/solr/collection1/conf/schema.xml.raw
+++ b/smartv/solr/collection1/conf/schema.xml
@@ -421,13 +421,13 @@
     <!-- mmseg4j-solr -->
     <fieldtype name="textComplex" class="solr.TextField" positionIncrementGap="100">
       <analyzer>
-         <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>           
+         <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/"/>      
       </analyzer>
     </fieldtype>
 
     <fieldtype name="textMaxWord" class="solr.TextField" positionIncrementGap="100">
       <analyzer type="index">
-        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
+        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic" />
       </analyzer>
       <analyzer type="query">
         <tokenizer class="solr.WhitespaceTokenizerFactory"/>
@@ -436,7 +436,7 @@
 
     <fieldtype name="textSimple" class="solr.TextField" positionIncrementGap="100">
       <analyzer>
-        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
+        <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" dicPath="/ccicall/opt/solr-5.5.1/smartv/solr/collection1/conf/smartv_dic/" />
       </analyzer>
     </fieldtype>

# 需要安装`lsof`组件
> su -
> yum -y install lsof
Loaded plugins: product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package lsof.x86_64 0:4.82-4.el6 will be installed
--> Finished Dependency Resolution
[...]
Installed:
  lsof.x86_64 0:4.82-4.el6                                                                                 

Complete!

# 为了解决`solr5.x`版本默认不能使用`iframe`的问题，需要替换核心包
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

# 替换核心包
> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar dist/
> md5sum dist/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  dist/solr-core-5.5.1.jar

> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar server/solr-webapp/webapp/WEB-INF/lib/
> md5sum server/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  server/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar

> scp -r root@10.0.3.45:/Archive/Project_Dadi/02.sources/solr嵌入包/solr-core-5.5.1.jar smartv/solr-webapp/webapp/WEB-INF/lib/
> md5sum smartv/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar
8c638461b56881c7d3a54505190cafcb  smartv/solr-webapp/webapp/WEB-INF/lib/solr-core-5.5.1.jar

# 启动服务
#!# 启动之前要为用户安装`JAVA`，若为安装请详见`JAVA`安装文档
> su - ccicall
> cd /ccicall/opt/solr-5.5.1
> pwd
/ccicall/opt/solr-5.5.1

> bin/solr restart -c -m 8g -d smartv/
Waiting up to 30 seconds to see Solr running on port 8983 [/]  
Started Solr server on port 8983 (pid=6681). Happy searching!
```

> 进入页面 `http://10.0.3.48:8983/solr/#/~cloud` 查看集群情况

![](media/14872110765844/14872231981877.jpg)

`-EOF-`



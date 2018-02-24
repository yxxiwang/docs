# 运维平台（smartv-op）程序部署文档

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 目录

[TOC]

## 部署环境说明

名称|版本|缩写|默认安装软件组
---|---|---|---
`RedHat-Enterprise-Linux`|`6.5`|`RHEL-6.5`|`Minimal`

### 需要额外资源信息

资源|作用|说明
---|---|---
操作系统`ISO`文件|部署软件服务器使用|

## 准备

### 软件服务器

> 部署`软件服务器`，请详见`软件服务器（Repository Server）部署文档`。

### 安装必要组件

> **注意：**必要组件一般使用`yum`安装，所以这里依赖`软件服务器（Repository Server）`，若没有搭建请详见`软件服务器（Repository Server）部署文档`。

#### 组件列表

名称|作用
---|---
`openssh-clients`|`传输工具（scp）`
`vim`|`编辑工具`
`git`|`版本管理工具`
`httpd`|`Web容器`
`php`|`程序开发语言`
`php-xml`|`程序扩展插件`
`php-pdo`|`程序扩展插件`
`php-mysql`|`程序扩展插件`

#### 安装方法

##### 一般组件安装（使用`yum`）

```bash
# 切换成为`root`用户
> su - 
Password: 

> yum -y install openssh-clients vim git httpd php php-xml php-pdo php-mysql
[...]
```

## 部署流程

```bash
# 切换用户
> su - ccicall
 
# 切换目录
> cd /home/ccicall
> pwd
/home/ccicall

# 获取程序
> scp root@10.0.3.45:/Archive/Project_Dadi/02.sources/各运行程序/smartv-op.zip ./
> ll smartv-op.zip 
-rw-r--r-- 1 ccicall ccicall 105715674 Feb 20 10:30 smartv-op.zip

# 解压
> unzip smartv-op.zip
[...]

# 检查
> ll /home/ccicall/smartv-op
total 112
drwxr-xr-x  6 ccicall ccicall  4096 Dec 30 10:09 app
drwxr-xr-x  2 ccicall ccicall  4096 Dec 30 10:09 bin
-rw-r--r--  1 ccicall ccicall  2810 Dec 30 10:09 composer.json
-rw-r--r--  1 ccicall ccicall 77913 Dec 30 10:09 composer.lock
drwxr-xr-x  3 ccicall ccicall  4096 Dec 30 10:09 deploy
-rw-r--r--  1 ccicall ccicall  2550 Dec 30 10:09 README.md
drwxr-xr-x  3 ccicall ccicall  4096 Dec 30 10:09 src
drwxr-xr-x 24 ccicall ccicall  4096 Dec 30 10:09 vendor
drwxr-xr-x  8 ccicall ccicall  4096 Dec 30 10:09 web

# 删除原有日志和缓存文件
> rm -rf smartv-op/app/logs/* smartv-op/app/cache/*

# 赋权给日志和缓存目录
> chmod -R 777 smartv-op/app/logs/ smartv-op/app/cache/

# 检查
> ll -d smartv-op/app/logs/ smartv-op/app/cache/
drwxrwxrwx 2 ccicall ccicall 4096 Jan  6 10:27 smartv-op/app/cache/
drwxrwxrwx 2 ccicall ccicall 4096 Feb 20 10:33 smartv-op/app/logs/

# 修改配置文件
> cd /home/ccicall/smartv-op
> pwd
/home/ccicall/smartv-op

# 备份`httpd`配置文件
> cp deploy/smartv_deploy/smartv_ops.conf deploy/smartv_deploy/smartv_ops.conf.raw
> ll deploy/smartv_deploy/smartv_ops.conf*
-rwxr-xr-x 1 ccicall ccicall 846 Dec 30 10:09 deploy/smartv_deploy/smartv_ops.conf
-rwxr-xr-x 1 ccicall ccicall 846 Feb 20 13:55 deploy/smartv_deploy/smartv_ops.conf.raw

# 修改`httpd`配置文件
# 修改配置文件 `第7行` 和 `第12行`，将 `/home/work/smartv-op/web` 修改为 `/home/ccicall/smartv-op/web`
> vim deploy/smartv_deploy/smartv_ops.conf 
```

```bash
# Warning: Modifying this file will break the Zend Server Administration Interface

Listen *:8081
NameVirtualHost *:8081
# do not allow override of this value for the UI's Vhost as it should always be off when generating
 non-html content such as dynamic images
<VirtualHost *:8081>
        DocumentRoot "/home/ccicall/smartv-op/web"
        DirectoryIndex index.php
        ServerName smartvdemo.cloud.centrin.com.cn
        ErrorLog /var/log/httpd/smartv-op_error.log

        <Directory "/home/ccicall/smartv-op/web">
                AllowOverride All
                Allow from All
                RewriteEngine On
                # Explicitly disable rewriting for front controllers
                RewriteRule ^app_dev.php - [L]
                RewriteRule ^app.php - [L]
                RewriteCond %{REQUEST_FILENAME} !-f
                # Change below before deploying to production
                RewriteRule ^(.*)$ app.php [QSA,L]
                #RewriteRule ^(.*)$ app_dev.php [QSA,L]
        </Directory>
</VirtualHost>
```

```bash
# 对比修改后文件内容
> git diff deploy/smartv_deploy/smartv_ops.conf.raw deploy/smartv_deploy/smartv_ops.conf
diff --git a/deploy/smartv_deploy/smartv_ops.conf.raw b/deploy/smartv_deploy/smartv_ops.conf
index 1305c00..6a2a309 100755
--- a/deploy/smartv_deploy/smartv_ops.conf.raw
+++ b/deploy/smartv_deploy/smartv_ops.conf
@@ -4,12 +4,12 @@ Listen *:8081
 NameVirtualHost *:8081
 # do not allow override of this value for the UI's Vhost as it should always be off when generatin
 <VirtualHost *:8081>
-       DocumentRoot "/home/work/smartv-op/web"
+       DocumentRoot "/home/ccicall/smartv-op/web"
        DirectoryIndex index.php
        ServerName smartvdemo.cloud.centrin.com.cn
        ErrorLog /var/log/httpd/smartv-op_error.log
 
-       <Directory "/home/work/smartv-op/web">
+       <Directory "/home/ccicall/smartv-op/web">
                AllowOverride All
                Allow from All
                RewriteEngine On
                
# 修改	`redis`配置文件
# 修改第82行：`snc_redis.clients.default.dsn`中的`IP地址`，修改为实际`redis`的`IP地址`
> vim app/config/config.yml 

# 修改`mysql`配置文件
# 修改第4行：`database_host`中的`HOSTNAME`，修改为实际的`HOSTNAME`或者`IP地址`
> vim app/config/parameters.yml

# 修改`zabbix`和`cloudera_manager`地址
# 分别修改第6行和第13行，分别将`zabbix_server.host`和`cloudera_manager.host`修改为实际的`IP地址`
> vim app/config/smartv_ops.yml

# 修改前端配置文件
# 分别修改第2行，第7行，第12行，第17行和第21行，根据它们说明修改为实际`IP地址`
> vim web/vendor/smartv/app/config/config.js

# 创建数据库
> mysql -uroot -proot123 -h10.0.3.45 < deploy/smartv_deploy/smartv_ops.sql 

# 检查
> mysql -uroot -proot123 -h10.0.3.45 -e 'show databases;'
+--------------------+
| Database           |
+--------------------+
[...]
| smartv_ops         |
+--------------------+

# 执行清楚缓存命令
# 在执行本步骤之前确保已经安装了如下组件（安装方法详见上文`安装方法`章节）
# php,php-xml,php-pdo,php-mysql
> php app/console cache:clear --env=prod --no-debug
Clearing the cache for the prod environment with debug false

# 切换用户
> su - 

# 修改`httpd`使用用户组
> pwd
/root

# 停止`httpd`服务
> service httpd stop
Stopping httpd:                                            [  OK  ]

# 备份`httpd`配置文件
> cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.raw
> ll /etc/httpd/conf/httpd.conf*
-rw-r--r-- 1 root root 34419 Feb 21 10:59 /etc/httpd/conf/httpd.conf
-rw-r--r-- 1 root root 34419 Feb 21 10:59 /etc/httpd/conf/httpd.conf.raw

# 修改`httpd`使用用户组
# 修改第243行，`Group`，将 `Group apache` 修改为 `Group ccicall`
> vim /etc/httpd/conf/httpd.conf

# 对比修改后文件内容
> git diff /etc/httpd/conf/httpd.conf.raw /etc/httpd/conf/httpd.conf
diff --git a/etc/httpd/conf/httpd.conf.raw b/etc/httpd/conf/httpd.conf
index df0a8d2..7b5246a 100644
--- a/etc/httpd/conf/httpd.conf.raw
+++ b/etc/httpd/conf/httpd.conf
@@ -240,7 +240,7 @@ Include conf.d/*.conf
 #  don't use Group #-1 on these systems!
 #
 User apache
-Group apache 
+Group ccicall
 
 ### Section 2: 'Main' server configuration
 #

> cd /home/
> ll -d ccicall/
drwx------. 7 ccicall ccicall 4096 Feb 20 14:11 ccicall/

# 为`ccicall`目录`用户组`和`其他用户`，增加`读`和`执行`权限
> chmod +rx ccicall/
> ll -d ccicall/
drwxr-xr-x. 7 ccicall ccicall 4096 Feb 20 14:11 ccicall/

# 将刚才修改过的`httpd`配置文件拷贝到`httpd`配置目录中
# 在执行本步骤之前确保已经安装了`httpd`
> cp /home/ccicall/smartv-op/deploy/smartv_deploy/smartv_ops.conf /etc/httpd/conf.d/
> ll /etc/httpd/conf.d/smartv_ops.conf 
-rwxr-xr-x 1 root root 852 Feb 20 14:32 /etc/httpd/conf.d/smartv_ops.conf

# 启动`httpd`服务
> service httpd restart
Stopping httpd:                                            [FAILED]
Starting httpd: httpd: Could not reliably determine the servers fully qualified domain name, using 10.0.3.48 for ServerName
                                                           [  OK  ]
```

> 可以访问页面`http://10.0.3.48:8081/`，账号|密码：`admin|admin`

`-EOF-`



#  HTTPD 2.4 安装文档

## 作者

**@author: `anxu@centrin.com.cn` || `axu.home@gmail.com`**

## 安装步骤

### 背景说明

> 由于现在环境中已经存在 `httpd 2.2` 同时存在很多依赖比如 `PHP, Zabbix, Cloudera` 等，如果删除 `2.2` 那么这些组件全部都需要重装。所以本次安装 `httpd 2.4` 主要是将某些应用迁移至 `2.4` 最后完成 `2.2` 和 `2.4` 共存。

![](assets/markdown-img-paste-20171010145621358.png)

### 获取软件源

```bash
# 将 Httpd24.zip 放入 repo 服务器的根目录中，并解压
#!# 注：该操作是在软件服务器上执行
> ll /Archive/
total 14052
drwxr-xr-x  2 root root     4096 Apr 18 10:17 Ant
drwxr-xr-x  3 root root     4096 Jul 13 17:53 CentOS
drwxr-xr-x. 4 root root     4096 Nov 25  2016 Cloudera-Enterprise
drwxr-xr-x  2 root root     4096 Apr 19 17:26 Elasticsearch
drwxr-xr-x  3 root root     4096 Oct  9 15:14 Httpd24
-rw-r--r--  1 root root 14307514 Oct 10 14:58 Httpd24.zip
drwxr-xr-x  2 root root     4096 Apr 19 10:17 Java
drwxr-xr-x  2 root root     4096 Apr 18 13:48 Maven
drwxr-xr-x  2 root root     4096 Apr 18 09:20 MongoDB
drwxr-xr-x  3 root root     4096 Feb 15  2017 Moosefs
drwxr-xr-x. 4 root root     4096 Nov 23  2016 RedHat-Enterprise-Linux
drwxr-xr-x  2 root root     4096 Jun 12 16:51 Redis
drwxr-xr-x  2 root root     4096 Nov 25  2016 Resources
drwxr-xr-x  2 root root     4096 Nov 24  2016 Scripts
drwxr-xr-x  2 root root     4096 Apr 19 16:39 Solr
drwxr-xr-x  2 root root     4096 Apr 24 14:51 Thrift
drwxr-xr-x  2 root root     4096 Apr 18 13:40 Tomcat
drwxr-xr-x  3 root root     4096 Feb 24  2017 Webtatic-PHP
drwxr-xr-x  2 root root     4096 Apr 19 14:39 Wildfly
drwxr-xr-x  4 root root     4096 Mar 15  2017 Zabbix

# 检查解压结果
> ll /Archive/Httpd24
total 8
drwxr-xr-x 6 root root 4096 Oct  9 15:14 epel-6Server
-rw-r--r-- 1 root root  369 Oct  9 15:14 httpd24.repo

# 将 .repo 文件拷贝到需要安装 httpd 2.4 服务器的 /etc/yum.repo.d/ 目录中
#!# 注：该操作是在需要安装 httpd 2.4 的服务器上执行
# 切换目录
> cd /etc/yum.repos.d
> ll
total 16
-rw-r--r-- 1 root root 336 Nov 28  2016 cm5.repo
-rw-r--r-- 1 root root 227 Nov 28  2016 rhel-6.5-media.repo
-rw-r--r-- 1 root root 218 Feb 24  2017 webtatic-php.repo
-rw-r--r-- 1 root root 448 Feb 24  2017 zabbix.repo

# 下载 .repo 文件
> wget http://10.0.3.49/Httpd24/httpd24.repo
[...]

# 检查
> ll
total 20
[...]
-rw-r--r-- 1 root root 369 Oct  9 15:14 httpd24.repo
[...]

# 检查 httpd 2.4 软件源是否加载成功
> yum repolist
[...]
repo id                                       repo name                                                                  status
[...]
epel-httpd24                                  httpd-2.4 scl                                                                 24
[...]

# 再次检查
> yum list | grep httpd24
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
httpd24.x86_64                          1-6.el6                     epel-httpd24
httpd24-apr.x86_64                      1.4.8-2.el6                 epel-httpd24
httpd24-apr-debuginfo.x86_64            1.4.8-2.el6                 epel-httpd24
httpd24-apr-devel.x86_64                1.4.8-2.el6                 epel-httpd24
httpd24-apr-util.x86_64                 1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-debuginfo.x86_64       1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-devel.x86_64           1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-ldap.x86_64            1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-mysql.x86_64           1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-nss.x86_64             1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-odbc.x86_64            1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-openssl.x86_64         1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-pgsql.x86_64           1.5.2-5.el6                 epel-httpd24
httpd24-apr-util-sqlite.x86_64          1.5.2-5.el6                 epel-httpd24
httpd24-build.x86_64                    1-6.el6                     epel-httpd24
httpd24-httpd.x86_64                    2.4.6-5.el6                 epel-httpd24
httpd24-httpd-debuginfo.x86_64          2.4.6-5.el6                 epel-httpd24
httpd24-httpd-devel.x86_64              2.4.6-5.el6                 epel-httpd24
httpd24-httpd-tools.x86_64              2.4.6-5.el6                 epel-httpd24
httpd24-mod_ldap.x86_64                 2.4.6-5.el6                 epel-httpd24
httpd24-mod_proxy_html.x86_64           1:2.4.6-5.el6               epel-httpd24
httpd24-mod_session.x86_64              2.4.6-5.el6                 epel-httpd24
httpd24-mod_ssl.x86_64                  1:2.4.6-5.el6               epel-httpd24
httpd24-runtime.x86_64                  1-6.el6                     epel-httpd24
```

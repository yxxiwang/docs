## CAE（Centrin Analytics Engine）部署文档

[TOC]

<!-- more -->

### 部署文档

#### 准备（Prerequisites）

##### 依赖（Denpendency）

- gcc（系统）
- gcc-c++（系统）
- zlib-devel（系统）
- bzip2-devel（系统）
- sqlite-devel（系统）
- Python 2.7
- pip
- Django==1.9.8
- jpype==0.6.2
- snownlp==0.12.3
- gensim==0.13.4
- django-rest-swagger==0.3.8
- djangorestframework==3.3.3
- HanLP 1.3.1
- Word2Vec Model

#### 安装（Installation）

##### 安装系统依赖

```bash
# 切换用户
> su -  
 
# 安装依赖包
> yum -y install gcc gcc-c++ zlib-devel bzip2-devel sqlite-devel
[...]
```

##### 安装`python2.7`

> **需要有`packages.zip`，并放到`/root/`目录中**

```bash
# 切换用户
> su -  

# 切换目录
> cd ~
> ll packages.zip 
-rw-r--r--. 1 root root 302967424 Mar  1 17:45 packages.zip

# 解压
> unzip packages.zip
[...]

# 检查
> cd packages
> ll
total 158516
-rw-r--r--. 1 root root   1595408 Mar  1 16:11 get-pip.py
-rw-r--r--. 1 root root 142249690 Mar  1 15:44 jdk-7u76-linux-x64.tar.gz
-rw-r--r--. 1 root root    147893 Mar  1 15:38 JPype1-0.6.2.tar.gz
-rw-r--r--. 1 root root   1230829 Mar  1 16:45 pyparsing-2.1.10.tar.gz
-rw-r--r--. 1 root root  17076672 Mar  1 17:57 Python-2.7.13.tgz
drwxr-xr-x. 2 root root      4096 Mar  1 17:29 requirements
-rw-r--r--. 1 root root       493 Mar  1 17:29 requirements.list

# 解压 python
> tar -zxvf Python-2.7.13.tgz
[...]

> cd Python-2.7.13
> ll
total 1000
-rw-r--r--.  1 1000 1000  10914 Dec 18 04:05 aclocal.m4
-rwxr-xr-x.  1 1000 1000  43940 Dec 18 04:05 config.guess
-rwxr-xr-x.  1 1000 1000  36350 Dec 18 04:05 config.sub
-rwxr-xr-x.  1 1000 1000 443361 Dec 18 04:05 configure
-rw-r--r--.  1 1000 1000 141651 Dec 18 04:05 configure.ac
drwxr-xr-x. 22 1000 1000   4096 Dec 18 04:05 Demo
drwxr-xr-x. 18 1000 1000   4096 Dec 18 04:12 Doc
drwxr-xr-x.  2 1000 1000   4096 Dec 18 04:05 Grammar
drwxr-xr-x.  2 1000 1000   4096 Dec 18 04:05 Include
-rwxr-xr-x.  1 1000 1000   7122 Dec 18 04:05 install-sh
drwxr-xr-x. 47 1000 1000  12288 Dec 18 04:05 Lib
-rw-r--r--.  1 1000 1000  12767 Dec 18 04:05 LICENSE
drwxr-xr-x. 11 1000 1000   4096 Dec 18 04:05 Mac
-rw-r--r--.  1 1000 1000  48384 Dec 18 04:05 Makefile.pre.in
drwxr-xr-x.  4 1000 1000   4096 Dec 18 04:05 Misc
drwxr-xr-x.  9 1000 1000   4096 Dec 18 04:05 Modules
drwxr-xr-x.  3 1000 1000   4096 Dec 18 04:05 Objects
drwxr-xr-x.  2 1000 1000   4096 Dec 18 04:05 Parser
drwxr-xr-x.  9 1000 1000   4096 Dec 18 04:05 PC
drwxr-xr-x.  2 1000 1000   4096 Dec 18 04:05 PCbuild
-rw-r--r--.  1 1000 1000  35082 Dec 18 04:05 pyconfig.h.in
drwxr-xr-x.  2 1000 1000   4096 Dec 18 04:05 Python
-rw-r--r--.  1 1000 1000  55664 Dec 18 04:05 README
drwxr-xr-x.  5 1000 1000   4096 Dec 18 04:05 RISCOS
-rw-r--r--.  1 1000 1000  99034 Dec 18 04:05 setup.py
drwxr-xr-x. 23 1000 1000   4096 Dec 18 04:05 Tools

# 配置
> ./configure --prefix=/usr/local --enable-unicode=ucs4 --with-zlib-dir=/usr/local/lib --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
[...]

# 编译
> make && make install
[...]

# 检查
> ll /usr/local/bin/python  
lrwxrwxrwx. 1 root root 7 Mar  1 18:02 /usr/local/bin/python -> python2

# 删除（逻辑）原有python
> mv /usr/bin/python /usr/bin/python.deleted

# 替换成2.7版本
> ln -s /usr/local/bin/python /usr/bin/python
> ll /usr/bin/python*
lrwxrwxrwx. 1 root root   21 Mar  1 18:05 /usr/bin/python -> /usr/local/bin/python
lrwxrwxrwx. 1 root root    6 Feb 16 18:07 /usr/bin/python2 -> python
-rwxr-xr-x. 2 root root 9032 Sep  4  2013 /usr/bin/python2.6
-rwxr-xr-x. 2 root root 9032 Sep  4  2013 /usr/bin/python.deleted

# 检查
> python --version
Python 2.7.13

#!# 修改`yum`使用的`python`版本，要不使用`yum`时，会报`No module named yum`错误
# 备份`yum`文件
> cp /usr/bin/yum /usr/bin/yum.raw
> ll /usr/bin/yum*
-rwxr-xr-x. 1 root root   801 Jan  9  2013 /usr/bin/yum
[...]
-rwxr-xr-x. 1 root root   801 Mar  1 18:08 /usr/bin/yum.raw

# 修改`/usr/bin/yum`文件
# 将第一行修改为：`#!/usr/bin/python2.6`
# 原来：
#    #!/usr/bin/python
# 修改后：
#    #!/usr/bin/python2.6
> vim /usr/bin/yum

# 对比修改后文件内容（若执行下面命令需要安装`git`）
> git diff /usr/bin/yum.raw /usr/bin/yum
diff --git a/usr/bin/yum.raw b/usr/bin/yum
index 7ccee31..eb07e24 100755
--- a/usr/bin/yum.raw
+++ b/usr/bin/yum
@@ -1,4 +1,4 @@
-#!/usr/bin/python
+#!/usr/bin/python2.6
 import sys
 try:
     import yum
     
# 检查
#!# 不报`No module named yum`错误
> yum list | more
[...]
```

##### 安装`java`

```bash
# 切换用户
> su - 

# 切换目录
> cd /root/packages
> pwd
/root/packages

# 解压
> tar -zxvf jdk-7u76-linux-x64.tar.gz
[...]

# 检查
> ll jdk1.7.0_76/
total 19772
drwxr-xr-x. 2 uucp 143     4096 Dec 19  2014 bin
-r--r--r--. 1 uucp 143     3339 Dec 19  2014 COPYRIGHT
drwxr-xr-x. 4 uucp 143     4096 Dec 19  2014 db
drwxr-xr-x. 3 uucp 143     4096 Dec 19  2014 include
drwxr-xr-x. 5 uucp 143     4096 Dec 19  2014 jre
drwxr-xr-x. 5 uucp 143     4096 Dec 19  2014 lib
-r--r--r--. 1 uucp 143       40 Dec 19  2014 LICENSE
drwxr-xr-x. 4 uucp 143     4096 Dec 19  2014 man
-r--r--r--. 1 uucp 143      114 Dec 19  2014 README.html
-rw-r--r--. 1 uucp 143      499 Dec 19  2014 release
-rw-r--r--. 1 uucp 143 19917352 Dec 19  2014 src.zip
-rw-r--r--. 1 uucp 143   110114 Dec 18  2014 THIRDPARTYLICENSEREADME-JAVAFX.txt
-r--r--r--. 1 uucp 143   173559 Dec 19  2014 THIRDPARTYLICENSEREADME.txt

# 移动目录
> mv /root/packages/jdk1.7.0_76/ /root/

# 检查
> ll -d /root/jdk1.7.0_76/
drwxr-xr-x. 8 uucp 143 4096 Dec 19  2014 /root/jdk1.7.0_76/

# 修改权限
> chown -R root:root /root/jdk1.7.0_76/
> ll -d /root/jdk1.7.0_76/
drwxr-xr-x. 8 root root 4096 Dec 19  2014 /root/jdk1.7.0_76/

# 备份`/etc/profile`文件
> cp /etc/profile /etc/profile.raw
> ll -d /etc/profile*
-rw-r--r--. 1 root root 1796 Aug 20  2013 /etc/profile
[...]
-rw-r--r--. 1 root root 1796 Mar  1 18:16 /etc/profile.raw

# 修改`/etc/profile`
# 在文件最后添加如下内容：
# export JAVA_HOME=/root/jdk1.7.0_76
# export PATH=${JAVA_HOME}/bin:${PATH}
# export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
> vim /etc/profile

# 对比修改后文件内容（若执行下面命令需要安装`git`）
> git diff /etc/profile.raw /etc/profile
diff --git a/etc/profile.raw b/etc/profile
index 1b82ae6..5147aba 100644
--- a/etc/profile.raw
+++ b/etc/profile
@@ -76,3 +76,8 @@ done
 
 unset i
 unset -f pathmunge
+
+export JAVA_HOME=/root/jdk1.7.0_76
+export PATH=${JAVA_HOME}/bin:${PATH}
+export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
+

# 使得修改生效
> source /etc/profile

# 检查
> java -version
java version "1.7.0_45"
OpenJDK Runtime Environment (rhel-2.4.3.3.el6-x86_64 u45-b15)
OpenJDK 64-Bit Server VM (build 24.45-b08, mixed mode)
```

##### 安装`python`扩展

```bash
# 切换用户
> su - 

# 切换目录
> cd /root/packages
> pwd
/root/packages

# 安装`pip`
> python get-pip.py --no-index --find-links=/root/packages/requirements
Collecting pip
Collecting setuptools
Collecting wheel
Collecting packaging>=16.8 (from setuptools)
Collecting appdirs>=1.4.0 (from setuptools)
Collecting six>=1.6.0 (from setuptools)
Collecting pyparsing (from packaging>=16.8->setuptools)
Installing collected packages: pip, six, pyparsing, packaging, appdirs, setuptools, wheel
Successfully installed appdirs-1.4.2 packaging-16.8 pip-9.0.1 pyparsing-2.1.10 setuptools-34.3.0 six-1.10.0 wheel-0.30.0a0

# 检查
> pip -V
pip 9.0.1 from /usr/local/lib/python2.7/site-packages (python 2.7)

# 通过`pip`安装扩展
> cd /root/packages
> pwd
/root/packages

# 执行安装命令
> pip install --no-index --find-links=/root/packages/requirements -r requirements.list 
Collecting boto==2.45.0 (from -r requirements.list (line 1))
Collecting bz2file==0.98 (from -r requirements.list (line 2))
Collecting Django==1.9.8 (from -r requirements.list (line 3))
Collecting django-filter==0.13.0 (from -r requirements.list (line 4))
Collecting django-grappelli==2.8.1 (from -r requirements.list (line 5))
Collecting django-material==0.8.0 (from -r requirements.list (line 6))
Collecting django-rest-swagger==0.3.8 (from -r requirements.list (line 7))
Collecting djangorestframework==3.3.3 (from -r requirements.list (line 8))
Collecting future==0.16.0 (from -r requirements.list (line 9))
Collecting gensim==0.13.4.1 (from -r requirements.list (line 10))
Collecting M2Crypto==0.25.1 (from -r requirements.list (line 11))
Collecting Markdown==2.6.6 (from -r requirements.list (line 12))
Collecting MySQL-python==1.2.5 (from -r requirements.list (line 13))
Collecting numpy==1.12.0 (from -r requirements.list (line 14))
Collecting psutil==4.3.0 (from -r requirements.list (line 15))
Collecting Pygments==2.1.3 (from -r requirements.list (line 16))
Collecting PyHive==0.2.1 (from -r requirements.list (line 17))
Collecting PyYAML==3.11 (from -r requirements.list (line 18))
Collecting requests==2.12.4 (from -r requirements.list (line 19))
Collecting sasl==0.2.1 (from -r requirements.list (line 20))
Collecting scipy==0.18.1 (from -r requirements.list (line 21))
Collecting simplejson==3.10.0 (from -r requirements.list (line 22))
Requirement already satisfied: six==1.10.0 in /usr/local/lib/python2.7/site-packages (from -r requirements.list (line 23))
Collecting smart-open==1.3.5 (from -r requirements.list (line 24))
Collecting snownlp==0.12.3 (from -r requirements.list (line 25))
Collecting sqlparse==0.2.2 (from -r requirements.list (line 26))
Collecting thrift==0.9.3 (from -r requirements.list (line 27))
Collecting thrift-sasl==0.2.1 (from -r requirements.list (line 28))
Collecting typing==3.5.2.2 (from -r requirements.list (line 29))
Installing collected packages: boto, bz2file, Django, django-filter, django-grappelli, django-material, djangorestframework, PyYAML, django-rest-swagger, future, requests, smart-open, scipy, numpy, gensim, typing, M2Crypto, Markdown, MySQL-python, psutil, Pygments, PyHive, sasl, simplejson, snownlp, sqlparse, thrift, thrift-sasl
Successfully installed Django-1.9.8 M2Crypto-0.25.1 Markdown-2.6.6 MySQL-python-1.2.5 PyHive-0.2.1 PyYAML-3.11 Pygments-2.1.3 boto-2.45.0 bz2file-0.98 django-filter-0.13.0 django-grappelli-2.8.1 django-material-0.8.0 django-rest-swagger-0.3.8 djangorestframework-3.3.3 future-0.16.0 gensim-0.13.4.1 numpy-1.12.0 psutil-4.3.0 requests-2.12.4 sasl-0.2.1 scipy-0.18.1 simplejson-3.10.0 smart-open-1.3.5 snownlp-0.12.3 sqlparse-0.2.2 thrift-0.9.3 thrift-sasl-0.2.1 typing-3.5.2.2

# 安装Jpype
> tar -zxvf pyparsing-2.1.10.tar.gz
> cd pyparsing-2.1.10
> python setup.py install 
[...]

> cd /root/packages 
> unzip JPype1-0.6.2.tar.gz 
> cd JPype1-0.6.2
> python setup.py install
[...]
Installed /usr/local/lib/python2.7/site-packages/JPype1-0.6.2-py2.7-linux-x86_64.egg
Processing dependencies for JPype1==0.6.2
Finished processing dependencies for JPype1==0.6.2
```

##### 整体检查

```bash
> pip list
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning.
appdirs (1.4.2)
boto (2.45.0)
bz2file (0.98)
Django (1.9.8)
django-filter (0.13.0)
django-grappelli (2.8.1)
django-material (0.8.0)
django-rest-swagger (0.3.8)
djangorestframework (3.3.3)
future (0.16.0)
gensim (0.13.4.1)
JPype1 (0.6.2)
M2Crypto (0.25.1)
Markdown (2.6.6)
MySQL-python (1.2.5)
numpy (1.12.0)
packaging (16.8)
pip (9.0.1)
psutil (4.3.0)
Pygments (2.1.3)
PyHive (0.2.1)
pyparsing (2.1.10)
PyYAML (3.11)
requests (2.12.4)
sasl (0.2.1)
scipy (0.18.1)
setuptools (34.3.0)
simplejson (3.10.0)
six (1.10.0)
smart-open (1.3.5)
snownlp (0.12.3)
sqlparse (0.2.2)
thrift (0.9.3)
thrift-sasl (0.2.1)
typing (3.5.2.2)
wheel (0.30.0a0)
```

#### 获取项目

##### 获取最新版本代码

> 因为要使用 `svn` , 若没有相关组件请自行安装。 

```bash
> svn export http://172.31.117.4/svn/cae/trunk/ cae

A    cae
A    cae/manage.py
A    cae/caeta_functional_tests
A    cae/caeta_functional_tests/__init__.py
A    cae/caeta_functional_tests/tests.py
[...]
A    cae/cae/serializable.py
A    cae/README.md
Exported revision 23.

# cae 目录为文档以后的 {project_root}
> cd cae; ls -l

total 16
-rw-r--r--   1 axu  staff  1888  1 13 22:42 README.md
drwxr-xr-x   8 axu  staff   272  1 18 11:23 cae
drwxr-xr-x   4 axu  staff   136  1 18 11:23 caeta_functional_tests
-rwxr-xr-x   1 axu  staff   246  1 13 21:21 manage.py
drwxr-xr-x  24 axu  staff   816  1 18 11:23 ta
```

> **注意：** `cae目录` 为文档以后的项目根目录 `{project_root}`

#### 配置（Configuration）

##### 配置 `HanLP` 目录

> 数据包下载地址：`root@10.0.3.49:/opt/hanlp_datas.zip`

```bash
# 修改 {project_root}/cae/settings.py 的 `HANLP_LIB_DIR` 配置项
# 修改为 `HanLP` 数据包目录
> cae axu$ cat cae/settings.py | grep HANLP_LIB_DIR

HANLP_LIB_DIR = '/Users/axu/code/axuProject/Learn/learn_python/00.fixtures/02.hanlp'
HANLP_CONF_DIR = HANLP_LIB_DIR  # 配置文件目录
HANLP_JAR_PATH = "%s/hanlp-1.3.1.jar" % HANLP_LIB_DIR  # jar包路径
```

##### 配置 `HanLP` 的配置文件（ `hanlp.properties` ）

> `hanlp.properties` 文件在 `{HANLP_LIB_DIR}` 中

```bash
# 修改 `hanlp.properties` 文件中的 `root` 配置，修改为 `{HANLP_LIB_DIR}`
> cat {HANLP_LIB_DIR}/hanlp.properties | grep root

#root=/Users/axu/code/axuProject/Learn/learn_python/00.fixtures/02.hanlp/
root=/Users/axu/Downloads/
```

##### 配置 `Word2Vec` 模型

> 默认的 `Word2Vec` 模型在 `HanLP` 的数据包中，数据包位置详见 <配置 `HanLP` 目录> 章节。

```bash
# 修改 {project_root}/cae/settings.py 的 `DEFAULT_WORD2VEC_MODEL` 配置项
# 修改为 `Word2Vec` 模型存放路径
> cat cae/settings.py | grep DEFAULT_WORD2VEC_MODEL

DEFAULT_WORD2VEC_MODEL = '/Users/axu/code/axuProject/Learn/learn_python/00.fixtures/03.word2vec/03.models/trained_model.bin'
```

#### 初始化数据库（Initialization Database）

> 因为项目的 `用户自定义词库` 是存储在 `数据库` 中，所以必须要配置数据库。
> 项目默认使用的数据库是 `sqllite` 。

```bash
# 进入项目根目录
> cd {project_root}
> ll

total 296
-rw-r--r--   1 axu  staff     246  1 13 14:45 README.md
drwxr-xr-x  13 axu  staff     442  1 13 13:40 cae
-rw-r--r--   1 axu  staff  135168  1 13 13:40 db.sqlite3
drwxr-xr-x   3 axu  staff     102  1 13 12:17 logs
-rwxr-xr-x   1 axu  staff     246  1 13 12:17 manage.py
-rw-r--r--   1 axu  staff    6477  1 13 12:17 nature_classify.list
drwxr-xr-x  39 axu  staff    1326  1 13 14:35 ta

# 执行初始化数据库命令
> python manage.py makemigrations

Migrations for ta:
  0001_initial.py:
    - Create model CustomDictionary

> python manage.py migrate

Operations to perform:
  Apply all migrations: admin, contenttypes, ta, auth, sessions
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying sessions.0001_initial... OK
  Applying ta.0001_initial... OK
```

#### 运行测试程序（Run Test）

```bash
> python manage.py test

Creating test database for alias 'default'...
.................................
----------------------------------------------------------------------
Ran 33 tests in 40.470s

OK
Destroying test database for alias 'default'...
```

`-EOF-`



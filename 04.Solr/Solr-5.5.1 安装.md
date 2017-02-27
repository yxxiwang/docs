# Solr-5.5.1 安装

## 目录
[TOC]

## master 
### 安装
1. `cp root@10.0.3.48:/Archive/software/solr-5.5.1-master.zip /opt/` 拷贝编译完成的包到服务器中

### 启动流程
1. `unzip solr-5.5.1-master.zip`
2. `cd /opt/solr-5.5.1`
3. `rm -rf bin/*.pid` 删除原有pid文件
4. `rm -rf smartv/solr/zoo_data/*` 删除原有zk数据文件
5. `mkdir -p /data/solr/collection1/` 创建索引存储文件目录
6. `bin/solr restart -c -m 8g -d smartv/` 启动命令
7. 登陆 http://10.0.3.48:8983 查看

## slave
### 安装
1. `cp root@10.0.3.48:/Archive/software/solr-5.5.1-slave.zip /opt/` 拷贝编译完成的包到服务器中

### 启动流程
1. `unzip solr-5.5.1-slave.zip`
2. `cd /opt/solr-5.5.1`
3. `rm -rf bin/*.pid` 删除原有pid文件
4. `rm -rf smartv/solr/zoo_data/*` 删除原有zk数据文件
5. `mkdir -p /data/solr/collection1/` 创建索引存储文件目录
6. 修改 `/opt/solr-5.5.1/bin/solr.in.sh` 文件中的 `ZK_HOST` 参数，修改为 `master` 节点信息
7. 查看修改后内容 `cat /opt/solr-5.5.1/bin/solr.in.sh | grep ZK_HOST`
8. `bin/solr restart -c -m 8g -d smartv/` 启动命令
9. 登陆 http://10.0.3.48:8983 查看

## 字段修改流程
1. 登陆 master 服务器
2. 修改 schema 文件，`/opt/solr-5.5.1/smartv/solr/collection1/conf/schema.xml` 添加 `field`，若想支持中文分词，`field.type` 必须设置为 `textMaxWord`。
3. 将修改后的 schema 文件，同步到所有服务器中。
4. 依次重启所有服务器，注：必须首先重启master服务器。重启命令 `cd /opt/solr-5.5.1;bin/solr restart -c -m 8g -d smartv/`

`-EOF-`




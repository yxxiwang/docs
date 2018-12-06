# 运维平台（smartv-op）nginx环境安装

## 作者

**@author: `anxu@centrin.com.cn` || `wangxi@163.com`**

## 目录

[TOC]

## 部署环境说明

### 软件服务器
> 部署`软件服务器`，请详见`软件服务器（Repository Server）部署文档`。

### 如果能连接外网
如果是centos6/redhat6环境平台：
wget http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
rpm -ivh nginx-release-centos-6-0.el6.ngx.noarch.rpm     
如果是centos7/redhat7环境平台：
wget http://nginx.org/packages/rhel/7/noarch/RPMS/nginx-release-rhel-7-0.el7.ngx.noarch.rpm
rpm -ivh nginx-release-rhel-7-0.el7.ngx.noarch.rpm

yum -y install nginx

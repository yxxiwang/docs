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

在/etc/nginx/conf.d/目录中，增加一个cdh.conf文件，内容如下。
该文件对cdh及hue做反向代理并消除X-Frame-Options头，允许进行跨域访问

``` javascript 
upstream cdh_servers {
    server 10.0.3.61:7180;    
}

upstream hue_servers {
    server 10.0.3.61:8888;    
}

server {
    listen   7180;

    access_log /var/log/nginx/solr-access.log ;
    error_log  /var/log/nginx/solr-error.log ;

    location / {
        proxy_redirect off ;
        proxy_hide_header X-Frame-Options; # 
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr:$remote_port;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 100m;
        client_body_buffer_size 50m;
        #client_body_temp_path /home/www/nginx_temp;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        #proxy_buffer_size 256k;
        proxy_buffer_size 50m;
        #proxy_buffers 4 256k;
        proxy_buffers 4 50m;
        proxy_busy_buffers_size 50m;
        #proxy_temp_file_write_size 256k;
        proxy_temp_file_write_size 50m;
        proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;
        proxy_max_temp_file_size 128m;
        proxy_pass    http://cdh_servers;
    }
}

server {
    listen   8888;

    access_log /var/log/nginx/solr-access.log ;
    error_log  /var/log/nginx/solr-error.log ;

    location / {
        #proxy_redirect on ;
        proxy_hide_header X-Frame-Options;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr:$remote_port  ;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 100m;
        client_body_buffer_size 50m;
        #client_body_temp_path /home/www/nginx_temp;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        #proxy_buffer_size 256k;
        proxy_buffer_size 50m;
        #proxy_buffers 4 256k;
        proxy_buffers 4 50m;
        proxy_busy_buffers_size 50m;
        #proxy_temp_file_write_size 256k;
        proxy_temp_file_write_size 50m;
        proxy_next_upstream error timeout invalid_header http_500 http_503 http_404;
        proxy_max_temp_file_size 128m;
        proxy_pass    http://hue_servers;
    }
}
```

完成后 执行 service nginx restart 命令对nginx服务进行重启
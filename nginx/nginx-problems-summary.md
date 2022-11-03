# nginx常见问题汇总

## 页面乱码解决方法

在server段里加以下两行

```shell script
default_type 'text/html';
charset utf-8;
```
然后重启就行了

```shell script
sudo nginx -s reload
```

## nginx  配置文件.*conf 编码的问题

* Linux：一般采用UTF-8-BOM编码  文本编辑时就可以保存
* Windows: 一般采用UTf-8编码 文本编辑时就可以保存

避免出现如下问题

```text
Nginx:[emerg] unknown directive “server”
invalid number of arguments in "root" directive  
```

如果出现，解决方式 `root后面那一行数据加分号`

## nginx报错 *441119082 client intended to send too large body: 1331696 bytes

### 1.在配置文件入口中的http段落中添加对文件上传的限制 `client_max_body_size 100m;`

```shell script
vim /usr/local/nginx/conf/nginx.conf
http {
    include       mime.types;
    server_tokens off;

    sendfile    on;
    tcp_nopush  on;
    tcp_nodelay on;
    keepalive_timeout  65;

    log_format  main  '$proxy_add_x_forwarded_for  $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" '
                      'upstream: $upstream_addr';
    access_log off;

    client_max_body_size 100m;
```

## 2.在配置文件中location段落中添加对文件上传的限制 `client_max_body_size 100m`

```shell script
vim /usr/local/nginx/conf/nginx.conf
location /group1/M00 {
    root   /data/fastdfs/data;
    include gzip.conf;
    ngx_fastdfs_module;
    client_max_body_size 100m;
    expires 12h;
}

location /group2/M00 {
    root   /data/fastdfs_group2/data;
    ngx_fastdfs_module;
    client_max_body_size 100m;
    #access_log /usr/local/nginx/logs/group2_pic.log main;
    expires 12h;
    include gzip.conf;
}
```

## nginx 安装时 make: 没有规则可以创建“default”需要的目标“build” 的解决方案

提前安装 zlib 和 pcr

## /usr/local/src/pcre-8.35/missing: line 81: aclocal-1.14: command not found

```shell script
autoreconf -ivf
```

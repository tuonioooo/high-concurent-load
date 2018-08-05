# nginx报错 client intended to send too large body: 1331696 bytes

### 1.nginx后台error日志报错 {#1nginx后台error日志报错}

```
2016/02/05 16:23:56 [error] 12024#0: *441106971 connect() failed (111: Connection refused) while connecting to upstream, client: 113.214.1.10, server: localhost, request: "GET /h5teb/ugcH5/index.htm?source=android&mall=8&TGC=911FDD2F99B84D528F0A7EE71780A943 HTTP/1.1", upstream: "http://113.214.1.23:8000/h5teb/ugcH5/index.htm?source=android&mall=8&TGC=911FDD2F99B84D528F0A7EE71780A943", host: "www.testcrm.com"
2016/02/05 16:48:14 [error] 12013#0: *441119082 client intended to send too large body: 1331696 bytes, client: 113.214.1.10, server: localhost, request: "POST /h5teb/complaints/save.htm HTTP/1.1", host: "www.testcrm.com", referrer: "http://www.testcrm.com/h5teb/complaints/index.htm"
```

### 2.web工程中添加对文件上传的限制  client\_max\_body\_size 100m;  {#2web工程中添加对文件上传的限制}

```
client_max_body_size 100m;
[nginx@wgq_idc_web_1_21 logs]$ vim /usr/local/nginx/conf/nginx.conf
http {
    include       mime.types;

    server_tokens off;

    sendfile        on;
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

### 3.静态资源中添加对文件上传的限制 {#3静态资源中添加对文件上传的限制}

```
[fastdfs@wgq_test_crm~]$  vim /usr/local/nginx/conf/nginx.conf
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




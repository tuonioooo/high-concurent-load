# nginx配置单个端口对应多个server

## 简单示例1

```text
upstream www.a.com {
    server localhost:8080;
}
upstream pan.a.com {
    server localhost:8081;
}
 
server {
    listen       80;
    server_name  www.a.com;
    location / {
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_pass http://www.a.com;
    }
}

server {
    listen       80;
    server_name  pan.a.com;
    location / {
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_pass http://pan.a.com;
    }
}

```

> 说明：  
> 一个80端口 对应两个server `www.a.com`、 `pan.a.com` 

## 简单示例2

```text
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {    client_max_body_size 100M;
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    #sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;

    upstream  tomcat8080_api {  
        server 127.0.0.1:8080  weight=1;
    }
    server {
        listen       80;
        server_name  后台接口域名;
        location / {
            proxy_pass http://tomcat8080_api;  
            proxy_redirect default;      
            #设置主机头和客户端真实地址，以便服务器获取客户端真实IP
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;            
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    
    upstream  tomcat8081_admin {  
        server 127.0.0.1:8081  weight=1;  
    }
    server {
        listen       80;
        server_name  管理员后台域名;
        location / {
            proxy_pass http://tomcat8081_admin;  
            proxy_redirect default;

        }
        #解决跨域
        location /apis  {                                # 自定义nginx接口前缀
            rewrite  ^/apis/(.*)$ /$1 break;            # 监听所有/apis前缀，是则转发后台api接口地址
            include  uwsgi_params;
            proxy_pass   http://127.0.0.1:8080;            # 后台api接口地址
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       80;
        server_name  管理员后台域名2;
        location / {
            proxy_pass http://tomcat8081_admin;  
            proxy_redirect default;
        }
        #解决跨域
        location /apis  {                                # 自定义nginx接口前缀
            rewrite  ^/apis/(.*)$ /$1 break;            # 监听所有/apis前缀，是则转发后台api接口地址
            include  uwsgi_params;
            proxy_pass   http://127.0.0.1:8080;            # 后台api接口地址
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server {
        listen       80;
        server_name  管理员后台域名3;
        location / {
            proxy_pass http://tomcat8081_admin;  
            proxy_redirect default;
        }
        #解决跨域
        location /apis  {                                # 自定义nginx接口前缀
            rewrite  ^/apis/(.*)$ /$1 break;            # 监听所有/apis前缀，是则转发后台api接口地址
            include  uwsgi_params;
            proxy_pass   http://127.0.0.1:8080;            # 后台api接口地址
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    
    upstream  tomcat8082_shanghu {  
        server 127.0.0.1:8082  weight=1;
    }
    server {
        listen       80;
        server_name  商家后台域名;
        location / {
            proxy_pass http://tomcat8082_shanghu;
            proxy_redirect default;
        }
        #解决跨域
        location /apis  {                                # 自定义nginx接口前缀
            rewrite  ^/apis/(.*)$ /$1 break;            # 监听所有/apis前缀，是则转发后台api接口地址
            include  uwsgi_params;
            proxy_pass   http://127.0.0.1:8080;            # 后台api接口地址
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

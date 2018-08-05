# windows下nginx安装、配置与使用

目前国内各大门户网站已经部署了Nginx，如新浪、网易、腾讯等；国内几个重要的视频分享网站也部署了Nginx，如六房间、酷6等。新近发现Nginx 技术在国内日趋火热，越来越多的网站开始部署Nginx。

```
相比apeach、iis，nginx以轻量级、高性能、稳定、配置简单、资源占用少等优势广受欢迎。
```

1\)下载地址：

[http://nginx.org](http://nginx.org/)

2\)启动

解压至c:\nginx，运行nginx.exe\(即nginx -c conf\nginx.conf\)，默认使用80端口，日志见文件夹C:\nginx\logs

3\)使用

[http://localhost](http://localhost/)

4\)关闭

nginx -s stop 或taskkill /F /IM nginx.exe &gt; nul

5\)常用配置

C:\nginx\conf\nginx.conf,使用自己定义的conf文件如my.conf，命令为nginx -c conf\my.conf

常用配置如下：

Nginx.conf代码

http {

server {

\#1.侦听80端口

listen 80;

location / {

\# 2. 默认主页目录在nginx安装目录的html子目录。

root html;

index index.html index.htm;

\# 3. 没有索引页时，罗列文件和子目录

autoindex on;

autoindex\_exact\_size on;

autoindex\_localtime on;

}

\# 4.指定虚拟目录

location /tshirt {

alias D:\programs\Apache2\htdocs\tshirt;

index index.html index.htm;

}

}

\# 5.虚拟主机www.emb.info配置

server {

listen 80;

server\_name www.emb.info;

access\_log emb.info/logs/access.log;

location / {

index index.html;

root emb.info/htdocs;

}

}

}

http {

server {

\#1.侦听80端口

listen 80;

location / {

\# 2. 默认主页目录在nginx安装目录的html子目录。

root html;

index index.html index.htm;

\# 3. 没有索引页时，罗列文件和子目录

autoindex on;

autoindex\_exact\_size on;

autoindex\_localtime on;

}

\# 4.指定虚拟目录

location /tshirt {

alias D:\programs\Apache2\htdocs\tshirt;

index index.html index.htm;

}

}

\# 5.虚拟主机www.emb.info配置

server {

listen 80;

server\_name www.emb.info;

access\_log emb.info/logs/access.log;

location / {

index index.html;

root emb.info/htdocs;

}

}

}

小提示：

运行nginx -V可以查看该Win32平台编译版支持哪些模块。我这里的结果为：

Log代码

nginx version: nginx/0.7.65

TLS SNI support enabled

configure arguments:

--builddir=objs.msvc8

--crossbuild=win32

--with-debug --prefix=

--conf-path=conf/nginx.conf

--pid-path=logs/nginx.pid

--http-log-path=logs/access.log

--error-log-path=logs/error.log

--sbin-path=nginx.exe

--http-client-body-temp-path=temp/client\_body\_temp

--http-proxy-temp-path=temp/proxy\_temp

--http-fastcgi-temp-path=temp/fastcgi\_temp

--with-cc-opt=-DFD\_SETSIZE=1024

--with-pcre=objs.msvc8/lib/pcre-7.9

--with-openssl=objs.msvc8/lib/openssl-0.9.8k

--with-openssl-opt=enable-tlsext

--with-zlib=objs.msvc8/lib/zlib-1.2.3

--with-select\_module

--with-http\_ssl\_module

--with-http\_realip\_module

--with-http\_addition\_module

--with-http\_sub\_module

--with-http\_dav\_module

--with-http\_stub\_status\_module

--with-http\_flv\_module

--with-http\_gzip\_static\_module

--with-http\_random\_index\_module

--with-http\_secure\_link\_module

--with-mail

--with-mail\_ssl\_module

--with-ipv6

nginx version: nginx/0.7.65

TLS SNI support enabled

configure arguments:

--builddir=objs.msvc8

--crossbuild=win32

--with-debug --prefix=

--conf-path=conf/nginx.conf

--pid-path=logs/nginx.pid

--http-log-path=logs/access.log

--error-log-path=logs/error.log

--sbin-path=nginx.exe

--http-client-body-temp-path=temp/client\_body\_temp

--http-proxy-temp-path=temp/proxy\_temp

--http-fastcgi-temp-path=temp/fastcgi\_temp

--with-cc-opt=-DFD\_SETSIZE=1024

--with-pcre=objs.msvc8/lib/pcre-7.9

--with-openssl=objs.msvc8/lib/openssl-0.9.8k

--with-openssl-opt=enable-tlsext

--with-zlib=objs.msvc8/lib/zlib-1.2.3

--with-select\_module

--with-http\_ssl\_module

--with-http\_realip\_module

--with-http\_addition\_module

--with-http\_sub\_module

--with-http\_dav\_module

--with-http\_stub\_status\_module

--with-http\_flv\_module

--with-http\_gzip\_static\_module

--with-http\_random\_index\_module

--with-http\_secure\_link\_module

--with-mail

--with-mail\_ssl\_module

--with-ipv6

显然，最经常用的memcache, rewrite模块都没在其中，因此该win32编译版本仅能供基本开发测试使用，对于产品平台，应该重新编译自己想要的win32版本，或者在linux下使用更方便。

6）查看nginx进程

tasklist /fi "imagename eq nginx.exe"，如下显示：

映像名称                       PID 会话名              会话\#       内存使用

========================= ======== ================ =========== ============

nginx.exe                     8944 Console                    1      5,128 K

nginx.exe                     6712 Console                    1      5,556 K

7）nginx常用命令

nginx -s stop 强制关闭

nginx -s quit 安全关闭

nginx -s reload 改变配置文件的时候，重启nginx工作进程，来时配置文件生效

nginx -s reopen 打开日志文件

8）其它

可以通过配置文件开启多个nginx工作进程，但同时只有其中一个nginx工作进程在工作，其他的阻塞等待。

一个nginx工作进程最多同时可以处理1024个连接。

nginx中需要共享内存的cache或者模块无法在windows下正常使用。

不过，nginx官方正在改进，将来nginx会以服务的方式运行，使用 I/O completion ports代替select方法，使多个工作进程能并发工作。

要使用nginx配合php-cgi使用，需要修改环境变量，否则，php-cgi运行一定次数就推出，需要重启，设置PHP\_FCGI\_MAX\_REQUESTS这个变量为0即可。

以上在win7上通过。

8）nginx以windows服务形式启动

1.下载微软两个工具：

instsrv.exe srvay.exe

2.执行命令:

instsrv Nginxc:/nginx/srvany.exe

3.配置Nginx的运行参数

可以直接将配置导入到注册表

\[HKEY\_LOCAL\_MACHINE/SYSTEM/CurrentControlSet/Services/NGINX/Parameters\]"Application"="C://nginx//nginx.exe""AppParameters"="""AppDirectory"="C://nginx//"

注意：windows 下的Nginx 内置的module 很多没有，用Nginx -V 命令查看。

9\)Nginx下部署mono+asp.net环境

1、从Mono for Windows中提取FastCGI-Mono-Server

2、Nginx nginx.conf 的配置：

| worker\_processes  1;error\_log  logs/error-debug.log info; events {    worker\_connections  1024;} http {    include       mime.types;    default\_type   text/plain;    sendfile        on;     keepalive\_timeout  65;    index  index.html index.htm;     server {        listen       80;        server\_name yourdomain.com;        index index.aspx default.aspx;         location / {          root   D:\www/yourwebapp;           fastcgi\_pass   127.0.0.1:8000;          fastcgi\_param  SCRIPT\_FILENAME  $document\_root/$fastcgi\_script\_name;          include       fastcgi\_params;       }    }} |
| :--- |


将上面的 FastCGI-Mono-Server 提取出来，所有文件全部注册到 GAC\(否则 Web 应用会找不到他们，当然你也可以直接放到

webapp/bin

\)，然后解压到某个文件夹，这里假设为 D:/FastCGI-Mono-Server。

之后我们就可以按下列命令运行 FastCGI：

fastcgi-mono-server2 /socket=tcp:127.0.0.1:8000 /root=

"

D:\www\yourwebapp

"

/applications=yourdomain.com:/:. /multiplex=True

最后执行运行 Nginx 服务器，我们的 ASP.Net 程序就能脱离 IIS。


# nginx超时时间设置

**client\_header\_timeout**

语法 client\_header\_timeout time  
默认值 60s  
上下文 http server  
说明 客户端向服务端发送一个完整的 request header 的超时时间。如果客户端在指定时间内没有发送一个完整的 request header，Nginx 返回 HTTP 408（Request Timed Out）。

> \# 配置段: http, server, location
>
> client\_header\_timeout 10s;

**client\_body\_timeout **

语法 client\_body\_timeout time  
默认值 60s  
上下文 http server location  
说明 指定客户端与服务端建立连接后发送 request body 的超时时间。如果客户端在指定时间内没有发送任何内容，Nginx 返回 HTTP 408（Request Timed Out）。

> \# 配置段: http, server, location
>
> client\_body\_timeout 20s;

**keepalive\_timeout （长连接类型）**

语法 keepalive\_timeout timeout \[ header\_timeout \]  
默认值 75s  
上下文 http server location  
说明 第一个参数指定了与client的keep-alive连接超时时间。服务器将会在这个时间后关闭连接。可选的第二个参数指定了在响应头Keep-Alive: timeout=time中的time值。这个头能够让一些浏览器主动关闭连接，这样服务器就不必要去关闭连接了。没有这个参数，nginx不会发送Keep-Alive响应头（尽管并不是由这个头来决定连接是否“keep-alive”）  
两个参数的值可并不相同

* 注意不同浏览器怎么处理“keep-alive”头
* MSIE和Opera忽略掉"Keep-Alive: timeout=&lt;N&gt;" header.
* MSIE保持连接大约60-65秒，然后发送TCP RST
* Opera永久保持长连接
* Mozilla keeps the connection alive for N plus about 1-10 seconds.
* Konqueror保持长连接N秒

> \# 配置段: http, server, location
>
> keepalive\_timeout 60s;

**lingering\_timeout**

语法 lingering\_timeout time  
默认值 5s  
上下文 http server location  
说明 lingering\_close生效后，在关闭连接前，会检测是否有用户发送的数据到达服务器，如果超过lingering\_timeout时间后还没有数据可读，就直接关闭连接；否则，必须在读取完连接缓冲区上的数据并丢弃掉后才会关闭连接。

> \# 配置段: http, server, location
>
> lingering\_timeout 60s;

**resolver**

语法: resolver address ... \[valid=time\];

默认值: —

配置段: http, server, location

配置DNS服务器IP地址。可以指定多个，以轮询方式请求。

nginx会缓存解析的结果。默认情况下，缓存时间是名字解析响应中的TTL字段的值，可以通过valid参数更改。

**resolver\_timeout**

语法 resolver\_timeout time  
默认值 30s  
上下文 http server location  
说明 该指令设置DNS解析超时时间

> server {
>
> listen 8090;
>
> location / {
>
> resolver 218.85.157.99 218.85.152.99;
>
> resolver\_timeout 30s;
>
> proxy\_pass [http://$host$request\_uri](http://$host$request_uri);
>
> }
>
> access\_log  /data/httplogs/proxy-$host-aceess.log;
>
> }

**proxy\_connect\_timeout**

语法 proxy\_connect\_timeout time  
默认值 60s  
上下文 http server location  
说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。  
这个不是等待后端返回页面的时间，那是由proxy\_read\_timeout声明的。如果你的upstream服务器起来了，但是hanging住了（例如，没有足够的线程处理请求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，由于与upstream服务器的连接已经建立了。

> //upstream配置
>
> upstream  my.upstream.com {
>
> server  10.1.1.2   max\_fails=1   fail\_timeout=10s;
>
> server  10.1.1.3  max\_fails=1   fail\_timeout=10s;   \#max\_fails默认值为1,fail\_timeout默认值为10s,max\_fails=0表示不做检查
>
> }
>
> //location配置
>
> location ~ / {
>
> proxy\_redirect off;
>
> proxy\_set\_header Host $host;
>
> proxy\_set\_header X-Real-IP $remote\_addr;
>
> proxy\_set\_header X-Forwarded-For $proxy\_add\_x\_forwarded\_for;
>
> proxy\_connect\_timeout  60s;   \#默认值60s, nginx连接到后端服务器的连接超时时间
>
> proxy\_timeout  10m; \#默认值为10分钟，nginx接收后端服务器的响应超时时间
>
> proxy\_pass      [http://my.upstream.com](http://my.upstream.com);
>
> }

**proxy\_read\_timeout**

语法 proxy\_read\_timeout time  
默认值 60s  
上下文 http server location  
说明 该指令设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。否则会报**504 Gateway Time-out问题**

> \# 配置段: http, server, location
>
> proxy\_read\_timeout 90;

**proxy\_send\_timeout**

语法 proxy\_send\_timeout time  
默认值 60s  
上下文 http server location  
说明 这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到新的数据，nginx会关闭连接

> \# 配置段: http, server, location
>
> **proxy\_send\_timeout** 90;

**proxy\_upstream\_fail\_timeout（fail\_timeout）**

语法 server address \[fail\_timeout=30s\]  
默认值 10s  
上下文 upstream  
说明 Upstream模块下 server指令的参数，设置了某一个upstream后端失败了指定次数（max\_fails）后，该后端不可操作的时间，默认为10秒

> upstream backend-others {
>
>             server 10.x.x.x max\_fails=0 fail\_timeout=10s;
>
>             server 10.x.x.x:81 max\_fails=0 fail\_timeout=10s;           
>
>             keepalive 20000;
>
>
>
>         }




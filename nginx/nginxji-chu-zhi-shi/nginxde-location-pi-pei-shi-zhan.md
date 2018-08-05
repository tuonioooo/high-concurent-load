# Nginx的location匹配实战

* ## location匹配的规则

请参考 [Nginx的location匹配规则](/nginx/nginxji-chu-zhi-shi/nginxde-location-pi-pei-gui-ze.md)

* ## location 匹配练习

### 1.先普通 location ，再正则 location

周边不少童鞋告诉我， nginx 是“先匹配正则 location 再匹配普通 location ”，其实这是一个误区， nginx 其实是“先匹配普通 location ，再匹配正则 location ”，但是普通 location 的匹配结果又分两种：一种是“严格精确匹配”，官方英文说法是“ exact match ”；另一种是“最大前缀匹配”，官方英文说法是“ Literal strings match the beginning portion of the query – the most specific match will be used. ”。我们做个实验：

**例1：**

```
server {
       listen       9090;
       server_name  localhost;
       location / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```

附录 nginx 的目录结构是： nginx-&gt;html-&gt;index.html

上述配置的意思是：

> location / {… deny all;} 普通 location 以“ / ”开始的 URI 请求（注意任何 HTTP 请求都必然以“/ ”开始，所以“ / ”的意思是所有的请求都能被匹配上），都拒绝访问； 
>
> location ~\.html$ {allow all;} 正则 location以 .html 结尾的 URI 请求，都允许访问。

**测试请求**：

```
[root@web108 ~]# curl http://localhost:9090/
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor=”white”>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```

```
[root@web108 ~]# curl http://localhost:9090/index.html
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body bgcolor=”white” text=”black”>
<center><h1>Welcome to nginx!</h1></center>
</body>
</html>
```

```
[root@web108 ~]# curl http://localhost:9090/index_notfound.html
<html>
<head><title>404 Not Found</title></head>
<body bgcolor=”white”>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```

```
[root@web108 ~]# curl http://localhost:9090/index.txt
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor=”white”>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```

**测试结果**：

| URI 请求 | HTTP 响应 |
| :--- | :--- |
| curl http://localhost:9090/ | 403 Forbidden |
| curl http://localhost:9090/index.html | Welcome to nginx! |
| curl http://localhost:9090/index\_notfound.html | 404 Not Found |
| curl http://localhost:9090/index.txt | 403 Forbidden |

**测试结果详解：**

> curl http://localhost:9090/ 的结果是“ 403 Forbidden ”，说明被匹配到“ location / {..deny all;} ”了，原因很简单HTTP 请求 GET / 被“严格精确”匹配到了普通 location / {} ，则会停止搜索正则 location ；
>
> curl http://localhost:9090/index.html 结果是“ Welcome to nginx! ”，说明没有被“ location / {…deny all;} ”匹配，否则会 403 Forbidden ，但 /index.html 的确也是以“ / ”开头的，只不过此时的普通 location / 的匹配结果是“最大前缀”匹配，所以 Nginx 会继续搜索正则 location ， location ~ \.html$ 表达了以 .html 结尾的都 allow all; 于是接着就访问到了实际存在的 index.html 页面。
>
> curl http://localhost:9090/index\_notfound.html   同样的道理先匹配 location / {} ，但属于“普通 location 的最大前缀匹配”，于是后面被“正则 location ” location ~ \.html$ {} 覆盖了，最终 allow all ； 但的确目录下不存在index\_notfound.html 页面，于是 404 Not Found 。
>
> curl http://localhost:9090/index.txt 因为先匹配上了 location / {..deny all;} 尽管属于“普通 location ”的最大前缀匹配结果，继续搜索正则 location ，但是 /index.txt 不是以 .html结尾的，正则 location 失败，最终采纳普通 location 的最大前缀匹配结果，于是 deny all 了。

### 2. 普通 location 的“隐式”严格匹配

**例题 2 **：**我们在例题 1 的基础上增加精确配置**

```
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```

**测试请求：**

```
[root@web108 ~]# curl http://localhost:9090/exact/match.html
<html>
<head><title>404 Not Found</title></head>
<body bgcolor=”white”>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.1.0</center>
</body>
</html>
```

**测试结果**

> 结果进一步验证了“普通 location ”的“严格精确”匹配会终止对正则 location 的搜索。这里我们小结下“普通 location”与“正则 location ”的匹配规则：先匹配普通 location ，再匹配正则 location ，但是如果普通 location 的匹配结果恰好是“严格精确（ exact match ）”的，则 nginx 不再尝试后面的正则 location ；如果普通 location 的匹配结果是“最大前缀”，则正则 location 的匹配覆盖普通 location 的匹配。也就是前面说的“正则 location 让步普通location 的严格精确匹配结果，但覆盖普通 location 的最大前缀匹配结果”。

### 3. 普通 location 的“显式”严格匹配和“ ^~ ” 前缀

上面我们演示的普通 location 都是不加任何前缀的，其实普通 location 也可以加前缀：“ ^~ ”和“ = ”。**其中“ ^~”的意思是“非正则，不需要继续正则匹配**”，也就是通常我们的普通 location ，还会继续搜索正则 location （恰好严格精确匹配除外），但是 nginx 很人性化允许配置人员告诉 nginx 某条普通 location ，无论最大前缀匹配，还是严格精确匹配都终止继续搜索正则 location ；而“ = ”则表达的是普通 location 不允许“最大前缀”匹配结果，必须严格等于，严格精确匹配。

**例题 3 ：“ ^~ ”前缀的使用**

```
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location ^~ / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```

把例题 2 中的 location / {} 修改成 location ^~ / {} ，再看看测试结果：

| URI 请求 | 修改前 | 修改后 |
| :--- | :--- | :--- |
| curl http://localhost:9090/ | 403 Forbidden | 403 Forbidden |
| curl http://localhost:9090/index.html | Welcome to nginx! | 403 Forbidden |
| curl http://localhost:9090/index\_notfound.html | 404 Not Found | 403 Forbidden |
| curl http://localhost:9090/exact/match.html | 404 Not Found | 404 Not Found |

除了 GET /exact/match.html 是 404 Not Found ，其余都是 403 Forbidden ，原因很简单所有请求都是以“ / ”开头，所以所有请求都能匹配上“ / ”普通 location ，但普通 location 的匹配原则是“最大前缀”，所以只有/exact/match.html 匹配到 location /exact/match.html {allow all;} ，其余都 location ^~ / {deny all;} 并终止正则搜索。

**例题 4 ：“ = ”前缀的使用**

```
server {
       listen       9090;
       server_name  localhost;
       location /exact/match.html {
           allow all;
       }
       location = / {
           root   html;
           index  index.html index.htm;
           deny all;
       }
       location ~ \.html$ {
           allow all;
       }
}
```

例题 4 相对例题 2 把 location / {} 修改成了 location = / {} ，再次测试结果：

| URI 请求 | 修改前 | 修改后 |
| :--- | :--- | :--- |
| curl http://localhost:9090/ | 403 Forbidden | 403 Forbidden |
| curl http://localhost:9090/index.html | Welcome to nginx! | Welcome to nginx! |
| curl http://localhost:9090/index\_notfound.html | 404 Not Found | 404 Not Found |
| curl http://localhost:9090/exact/match.html | 404 Not Found | 404 Not Found |
| curl http://localhost:9090/test.jsp | 403 Forbidden | 404 Not Found |

最能说明问题的测试是 GET /test.jsp ，实际上 /test.jsp 没有匹配正则 location （ location ~\.html$ ），也没有匹配 location = / {} ，如果按照 location / {} 的话，会“最大前缀”匹配到普通 location / {} ，结果是 deny all 。

### 4. 正则 location 与编辑顺序

location 的指令与编辑顺序无关，这句话不全对。对于普通 location 指令，匹配规则是：最大前缀匹配（与顺序无关），如果恰好是严格精确匹配结果或者加有前缀“ ^~ ”或“ = ”（符号“ = ”只能严格匹配，不能前缀匹配），则停止搜索正则 location ；但对于正则 location 的匹配规则是：按编辑顺序逐个匹配（与顺序有关），只要匹配上，就立即停止后面的搜索。

**配置 3.1**

```
server {
       listen       9090;
       server_name  localhost;
       location ~ \.html$ {
           allow all; 
       }  
       location ~ ^/prefix/.*\.html$ {
           deny all;  
       }  
}
```

**配置 3.2**

```
server {
       listen       9090;
       server_name  localhost;
       location ~ ^/prefix/.*\.html$ {
           deny all;  
       }  
       location ~ \.html$ {
           allow all; 
       } 
}
```

**测试结果：**

| URI 请求 | 配置 3.1 | 配置 3.2 |
| :--- | :--- | :--- |
| curl http://localhost:9090/regextest.html | 404 Not Found | 404 Not Found |
| curl http://localhost:9090/prefix/regextest.html | 404 Not Found | 403 Forbidden |

解释：

> Location ~ ^/prefix/.\*\.html$ {deny all;} 表示正则 location 对于以 /prefix/ 开头， .html 结尾的所有 URI 请求，都拒绝访问；   
>
> location ~\.html${allow all;} 表示正则 location 对于以 .html 结尾的 URI 请求，都允许访问。 实际上，prefix 的是 ~\.html$ 的子集。
>
> 在“配置 3.1 ”下，两个请求都匹配上 location ~\.html$ {allow all;} ，并且停止后面的搜索，于是都允许访问， 404 Not Found ；在“配置 3.2 ”下， /regextest.html 无法匹配 prefix ，于是继续搜索 ~\.html$ ，允许访问，于是 404 Not Found ；然而 /prefix/regextest.html 匹配到 prefix ，于是 deny all ， 403 Forbidden 。

**配置 3.3**

```
server {
       listen       9090;
       server_name  localhost;
       location  /prefix/ {
               deny all;  
       }  
       location  /prefix/mid/ {
               allow all; 
       }  
}
```

**配置 3.4**

```
server {
       listen       9090;
       server_name  localhost;
       location  /prefix/mid/ {
               allow all; 
       }  
       location  /prefix/ {
               deny all;  
       }  
}
```

**测试结果：**

| URI 请求 | 配置 3.1 | 配置 3.2 |
| :--- | :--- | :--- |
| curl http://localhost:9090/prefix/t.html | 403 Forbidden | 403 Forbidden |
| curl http://localhost:9090/prefix/mid/t.html | 404 Not Found | 404 Not Found |

测试结果表明：普通 location 的匹配规则是“最大前缀”匹配，而且与编辑顺序无关。

### \#5 “@” 前缀 Named Location 使用

REFER:  [http://nginx.org/en/docs/http/ngx\_http\_core\_module.html\#error\_page](http://nginx.org/en/docs/http/ngx_http_core_module.html#error_page)

假设配置如下：

```
server {
       listen       9090;
       server_name  localhost;
        location  / {
           root   html;
           index  index.html index.htm;
           allow all;
       }
       
#error_page 404 http://www.baidu.com # 直接这样是不允许的
       # 直接这样是不允许的
       error_page 404 = @fallback;
       location @fallback {
           proxy_pass http://www.baidu.com;
       }
}

```



上述配置文件的意思是：如果请求的 URI 存在，则本 nginx 返回对应的页面；如果不存在，则把请求代理到baidu.com 上去做个弥补（注： nginx 当发现 URI 对应的页面不存在， HTTP\_StatusCode 会是 404 ，此时error\_page 404 指令能捕获它）。

测试一：

\[root@web108 ~\]\# 

curl http://localhost:9090/nofound.html -i

HTTP/1.1 302 Found

Server: nginx/1.1.0

Date: Sat, 06 Aug 2011 08:17:21 GMT

Content-Type: text/html; charset=iso-8859-1

Location: http://localhost:9090/search/error.html

Connection: keep-alive

Cache-Control: max-age=86400

Expires: Sun, 07 Aug 2011 08:17:21 GMT

Content-Length: 222

&lt;

!DOCTYPE HTML PUBLIC “-//IETF//DTD HTML 2.0//EN”

&gt;

&lt;

html

&gt;

&lt;

head

&gt;

&lt;

title

&gt;

302 Found

&lt;

/title

&gt;

&lt;

/head

&gt;

&lt;

body

&gt;

&lt;

h1

&gt;

Found

&lt;

/h1

&gt;

&lt;

p

&gt;

The document has moved 

&lt;

a href=”http://www.baidu.com/search/error.html”

&gt;

here

&lt;

/a

&gt;

.

&lt;

/p

&gt;

&lt;

/body

&gt;

&lt;

/html

&gt;

\[root@web108 ~\]\#

当我们 GET /nofound.html 发送给本 nginx ， nginx 找不到对应的页面，于是 error\_page 404 = @fallback ，请求被代理到 

[http://www.baidu.com](http://www.baidu.com/)

 ，于是 nginx 给 http://www.baidu.com 发送了 GET /nofound.html ，但/nofound.html 页面在百度也不存在，百度 302 跳转到错误页。

直接访问 

[http://www.baidu.com/nofound.html](http://www.baidu.com/nofound.html)

 结果：

\[root@web108 ~\]\# curl http://www.baidu.com/nofound.html -i

HTTP/1.1 302 Found

Date: Sat, 06 Aug 2011 08:20:05 GMT

Server: Apache

Location: http://www.baidu.com/search/error.html

Cache-Control: max-age=86400

Expires: Sun, 07 Aug 2011 08:20:05 GMT

Content-Length: 222

Connection: Keep-Alive

Content-Type: text/html; charset=iso-8859-1

&lt;

!DOCTYPE HTML PUBLIC “-//IETF//DTD HTML 2.0//EN”

&gt;

&lt;

html

&gt;

&lt;

head

&gt;

&lt;

title

&gt;

302 Found

&lt;

/title

&gt;

&lt;

/head

&gt;

&lt;

body

&gt;

&lt;

h1

&gt;

Found

&lt;

/h1

&gt;

&lt;

p

&gt;

The document has moved 

&lt;

a href=”http://www.baidu.com/search/error.html”

&gt;

here

&lt;

/a

&gt;

.

&lt;

/p

&gt;

&lt;

/body

&gt;

&lt;

/html

&gt;

\[root@web108 ~\]\#

测试二：访问一个 nginx 不存在，但 baidu 存在的页面

\[root@web108 ~\]\# curl http://www.baidu.com/duty/ -i

HTTP/1.1 200 OK

Date: Sat, 06 Aug 2011 08:21:56 GMT

Server: Apache

P3P: CP=” OTI DSP COR IVA OUR IND COM ”

P3P: CP=” OTI DSP COR IVA OUR IND COM ”

Set-Cookie: BAIDUID=5C5D2B2FD083737A0C88CA7075A6601A:FG=1; expires=Sun, 05-Aug-12 08:21:56 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1

Set-Cookie: BAIDUID=5C5D2B2FD083737A2337F78F909CCB90:FG=1; expires=Sun, 05-Aug-12 08:21:56 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1

Last-Modified: Wed, 05 Jan 2011 06:44:53 GMT

ETag: “d66-49913b8efe340″

Accept-Ranges: bytes

Content-Length: 3430

Cache-Control: max-age=86400

Expires: Sun, 07 Aug 2011 08:21:56 GMT

Vary: Accept-Encoding,User-Agent

Connection: Keep-Alive

Content-Type: text/html

&lt;

!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN”

“http://www.w3.org/TR/html4/loose.dtd”

&gt;

。。。。

&lt;

/body

&gt;

&lt;

/html

&gt;

显示，的确百度这个页面是存在的。

\[root@web108 ~\]\#

 curl http://localhost:9090/duty/ -i

HTTP/1.1 200 OK

Server: nginx/1.1.0

Date: Sat, 06 Aug 2011 08:23:23 GMT

Content-Type: text/html

Connection: keep-alive

P3P: CP=” OTI DSP COR IVA OUR IND COM ”

P3P: CP=” OTI DSP COR IVA OUR IND COM ”

Set-Cookie: BAIDUID=8FEF0A3A2C31D277DCB4CC5F80B7F457:FG=1; expires=Sun, 05-Aug-12 08:23:23 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1

Set-Cookie: BAIDUID=8FEF0A3A2C31D277B1F87691AFFD7440:FG=1; expires=Sun, 05-Aug-12 08:23:23 GMT; max-age=31536000; path=/; domain=.baidu.com; version=1

Last-Modified: Wed, 05 Jan 2011 06:44:53 GMT

ETag: “d66-49913b8efe340″

Accept-Ranges: bytes

Content-Length: 3430

Cache-Control: max-age=86400

Expires: Sun, 07 Aug 2011 08:23:23 GMT

Vary: Accept-Encoding,User-Agent

&lt;

!DOCTYPE HTML PUBLIC “-//W3C//DTD HTML 4.01 Transitional//EN”

“http://www.w3.org/TR/html4/loose.dtd”

&gt;

&lt;

html

&gt;

。。。

&lt;

/body

&gt;

&lt;

/html

&gt;

当 curl http://localhost:9090/duty/ -i 时， nginx 没找到对应的页面，于是 error\_page = @fallback ，把请求代理到 baidu.com 。注意这里的 error\_page = @fallback 不是靠重定向实现的，而是所说的“ internally redirected （forward ）”。


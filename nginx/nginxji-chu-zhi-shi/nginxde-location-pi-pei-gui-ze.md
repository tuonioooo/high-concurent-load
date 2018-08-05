# Nginx的location匹配规则 

* ## **语法规则**

官方文档解释：[http://nginx.org/en/docs/http/ngx\_http\_core\_module.html\#location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)

location \[=\|~\|~\*\|^~\] /uri/ { … }

| 模式 | 含义 |
| :--- | :--- |
| location = /uri | = 表示_**精确匹配**_，只有完全匹配上才能生效 |
| location ^~ /uri | ^~ 开头对URL路径进行_**前缀匹配**_，并且在正则之前。 |
| location ~ pattern | 开头表示_**区分大小写的正则匹配**_ |
| location ~\* pattern | 开头表示_**不区分大小写的正则匹配**_ |
| location /uri | 不带任何修饰符，也表示前缀匹配，但是在正则匹配之后 |
| location / | 通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中的default |




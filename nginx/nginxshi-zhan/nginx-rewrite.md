# nginx proxy_pass 路径匹配、URL 重写


## 需要注意 proxy_pass 后跟的服务器 URL 是否以 / 结尾。

举例：Nginx 收到 URL 请求 `https://nginx_server_name/hello/world`，设置的代理路径为 `/hello/`（即在 location /hello/ 内设置代理）

不以 / 结尾的被代理服务器收到的请求路径是 /hello/world  
以 / 结尾的被代理服务器收到的请求路径是 /world  

如果是为了在同一个域名下以不同路径分配不同的 APP 应选择后者以 / 结尾 

Nginx 的代理 HTTP 版本默认为 1.0，若要设置为 1.1 则需添加下述配置：

```shell script
proxy_http_version 1.1;
proxy_set_header Connection "";
```

## URL 重写

以下示例： 用来域名重定向、强制 HTTPS。

```shell script
if ($host != 'lkxed.cn' ) {
  rewrite ^/(.*)$ https://lkxed.cn/$1 permanent;
}
```
> 说明：  
> 其中地址末尾的 $1 是前面 ^/(.*)$ 匹配到的第一个分组，也就是域名后跟的完整路径字符串。当我的重写规则这样配置了之后，若请求路径为 `http://www.lkxed.cn/app/hello/world`，则它会被重定向到 `https://lkxed.cn/app/hello/world`。一般都会在目标地址末尾加上 $1，作用是不使路径丢失。


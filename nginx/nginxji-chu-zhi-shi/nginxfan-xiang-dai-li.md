# nginx反向代理

就普通的反向代理来讲

Nginx的配置还是比较简单的，如：

```
 
location ~ /*{
proxy_pass http://127.0.0.1:8008;
}

```

或者可以

```
location /
{
proxy_pass http://127.0.0.1:8008;
}
```




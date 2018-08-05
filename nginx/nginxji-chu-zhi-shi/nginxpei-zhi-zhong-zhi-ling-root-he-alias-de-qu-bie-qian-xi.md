# Nginx配置中指令root和alias的区别浅析

* ## 前言

root和alias都可以定义在location模块中，都是用来指定请求资源的真实路径

* ## root

root配置的location，资源的**真实路径是root指定的值加上location指定的值，**比如：

```
location /i/ {
  root /data/w3;
}
```

请求 `http://foofish.net/i/top.gif `这个地址时，那么在服务器里面对应的真正的资源是` /data/w3/i/top.gif`文件

**注意：真实的路径是root指定的值加上location指定的值**

* ## alias 

alias指定的路径是location的别名，不管location的值怎么写，资源的 **真实路径都是 alias 指定的路径** ，比如：

```
location /i/ {
  alias /data/w3/;
}
```

同样请求 http://foofish.net/i/top.gif 时，在服务器查找的资源路径是： /data/w3/top.gif

* ## **其他区别：**

1.     1、 alias 只能作用在location中，而root可以存在server、http和location中。 
2.      2、alias 后面必须要用 “/” 结束，否则会找不到文件，而 root 则对 ”/” 可有可无。




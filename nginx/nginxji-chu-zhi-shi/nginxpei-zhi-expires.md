# nginx配置expires

* ## expires语法

语法：expires \[time\|epoch\|max\|off\]

默认值：expires off

作用域：http, server, location

使用本指令可以控制HTTP应答中的“Expires”和“Cache-Control”的头标，（起到控制页面缓存的作用）。

可以在time值中使用正数或负数。“Expires”头标的值将通过当前系统时间加上您设定的 time 值来获得。

* ## 参数说明

> epoch 指定“Expires”的值为 1 January, 1970, 00:00:01 GMT。
>
> max 指定“Expires”的值为 31 December 2037 23:59:59 GMT，“Cache-Control”的值为10年。
>
> -1 指定“Expires”的值为 服务器当前时间 -1s,即永远过期
>
> “Cache-Control”头标的值由您指定的时间来决定：
>
> * 负数：
>   Cache-Control: no-cache
> * 正数或零：
>   Cache-Control: max-age = \#, \# 
>   为您指定时间的秒数。
>
> "off" 表示不修改“Expires”和“Cache-Control”的值

示例:

控制图片等过期时间为30天，当然这个时间可以设置的更长。具体视情况而定比如

```
location~ \.(gif|jpg|jpeg|png|bmp|ico)$ {
           expires 30d;
}
```

控制匹配/resource/或者/mediatorModule/里所有的文件缓存设置到最长时间比如

```
location ~ /(resource|mediatorModule)/ {
     root    /opt/demo;
     expires max;
}
```




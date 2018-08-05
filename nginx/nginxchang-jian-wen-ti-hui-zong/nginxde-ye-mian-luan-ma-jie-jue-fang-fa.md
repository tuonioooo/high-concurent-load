# Nginx的页面乱码解决方法

在server段里加以下两行

```
default_type 'text/html';
charset utf-8;
```

然后重启就行了

```
sudo nginx -s reload
```




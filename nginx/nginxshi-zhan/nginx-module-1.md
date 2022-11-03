# nginx 在已安装的情况下 安装http_image_filter_module

## 简介 

`nginx_http_image_filter_module` 在nginx 0.7.54以后才出现的，用于对JPEG, GIF和PNG图片进行转换处理 这个模块默认不被编译，所以要在编译nginx源码的时候，加入相关配置信息

在配置之前先安装gd-devel ，HttpImageFilterModule模块需要依赖gd-devel的支持，可以使用yum或apt-get方便地安装，如果未安装回报“/configure: error: the HTTP image filter module requires the GD library.”错误

参考:

[文档1](https://blog.csdn.net/hb1707/article/details/52510611?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
[文档2](https://blog.csdn.net/qq_36663951/article/details/80988392)

## 安装

```shell script
yum install gd-devel
或
apt-get install libgd2-xpm libgd2-xpm-dev
```


1.查看原有的nginx配置命令

```shell script
$ /usr/local/nginx/sbin/nginx -V
```

2.进入之前下载的源码目录，没有了，重新下载，我的目前是nginx-1.17.5版本

```shell script
$ cd /usr/local/src/nginx-1.17.5
```

3.进入目录后执行如下命令

```shell script
$ ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module --with-pcre=/usr/local/src/pcre-8.35 ***
$ make
```

> 不需要make install ，在已有安装的情况下编译即可，其中 `***` 是 安装nginx原始的配置请务必加上 可以参考 [nginx安装教程](../nginx-setup.md)

4.备份原有目录下

```shell script
$ mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
```

5.将编译好的nginx复制到已安装nginx的sbin目录

```shell script
$ cp objs/nginx /usr/local/nginx/sbin/
```

6.查看nginx 安装模块

```shell script
$ /usr/local/nginx/sbin/nginx -V
```

# windows下nginx启动、停止、重名命令

Windows下Nginx的启动、停止等命令

在Windows下使用Nginx，我们需要掌握一些基本的操作命令，比如：启动、停止Nginx服务，重新载入Nginx等，下面我就进行一些简单的介绍。

1、启动：

C:\server\nginx-1.0.2

&gt;

start nginx

或

C:\server\nginx-1.0.2

&gt;

nginx.exe

注：建议使用第一种，第二种会使你的cmd窗口一直处于执行中，不能进行其他命令操作。

2、停止：

C:\server\nginx-1.0.2

&gt;

nginx.exe -s stop

或

C:\server\nginx-1.0.2

&gt;

nginx.exe -s quit

  


注：stop是快速停止nginx，可能并不保存相关信息；quit是完整有序的停止nginx，并保存相关信息。

3、重新载入Nginx：

C:\server\nginx-1.0.2

&gt;

nginx.exe -s reload

当配置信息修改，需要重新载入这些配置时使用此命令。

4、重新打开日志文件：

C:\server\nginx-1.0.2

&gt;

nginx.exe -s reopen

5、查看Nginx版本：

C:\server\nginx-1.0.2

&gt;

nginx -v

  


  


常用的 Nginx 参数和控制

       程序运行参数Nginx 安装后只有一个程序文件，本身并不提供各种管理程序，它是使用参数和系统信号机制对 Nginx 进程本身进行控制的。 Nginx 的参数包括有如下几个：

-c 

&lt;

path\_to\_config

&gt;

：使用指定的配置文件而不是 conf 目录下的 nginx.conf 。

-t：测试配置文件是否正确，在运行时需要重新加载配置的时候，此命令非常重要，用来检测所修改的配置文件是否有语法错误。

-v：显示 nginx 版本号。

-V：显示 nginx 的版本号以及编译环境信息以及编译时的参数。


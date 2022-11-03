# nginx注册系统服务

## nginx注册到 `Centos6` 系统服务

### 简介：

Nginx安装完成后默认不会注册为系统服务，所以需要手工添加系统服务脚本。在/etc/init.d目录下新建nginx文件，并更改权限其即可。

### 1、新建nginx文件

1.1、新建文件：

```shell script
vi /etc/init.d/nginx
```

1.2、添加内容如下

```shell script
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description: Nginx is an HTTP(S) server, HTTP(S) reverse  
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
# 这里要根据实际情况修改
nginx="/usr/local/nginx/nginx"
prog=$(basename $nginx)
# 这里要根据实际情况修改
NGINX_CONF_FILE="/usr/local/nginx/nginx.conf"
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
lockfile=/var/lock/subsys/nginx
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
    killall -9 nginx
}
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
force_reload() {
    restart
}
configtest() {
    $nginx -t -c $NGINX_CONF_FILE
}
rh_status() {
    status $prog
}
rh_status_q() {
    rh_status >/dev/null 2>&1
}
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
        ;;
    *)    
      echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

### 2、修改其权限并开机启动

2.1、修改权限：

```shell script
chmod 755 /etc/init.d/nginx
```

2.2、开机启动：

```shell script
chkconfig nginx on
```

2.3、查看开机启动的服务：

```shell script
chkconfig --list*
```
### 3、启动、停止、重启命令

3.1、启动服务：

```shell script
service nginx start
```

3.2、停止服务：

```shell script
service nginx stop
```

3.3、重启服务：

```shell script
service nginx reload
```


## nginx注册到 `Centos7` 系统服务

### 创建nginx启动命令脚本

```shell script
vi /etc/init.d/nginx
```

### 插入以下内容, 注意修改PATH和NAME字段, 匹配自己的安装路径

```shell script
#! /bin/bash
# chkconfig: - 85 15
PATH=/usr/local/nginx
DESC="nginx daemon"
NAME=nginx
DAEMON=$PATH/sbin/$NAME
CONFIGFILE=$PATH/conf/$NAME.conf
PIDFILE=$PATH/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
set -e
[ -x "$DAEMON" ] || exit 0
do_start() {
$DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
do_stop() {
$DAEMON -s stop || echo -n "nginx not running"
}
do_reload() {
$DAEMON -s reload || echo -n "nginx can't reload"
}
case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac
exit 0
```

> 注意：上面 文件最好是在终端服务器上编辑，否则会产生其他编码，或者其他不可预测额问题：比如如下问题：
>
> Failed at step EXEC spawning /etc/rc.d/init.d/nginx: No such file or directory
>
> ExecStart=/etc/rc.d/init.d/nginx start \(code=exited, status=203/EXEC\)

### 设置执行权限

```shell script
chmod a+x /etc/init.d/nginx
```

### 注册成服务

```shell script
chkconfig --add nginx
```

### 设置开机启动

```shell script
chkconfig nginx on
```

### 重启, 查看nginx服务是否自动启动

```shell script
shutdown -h 0 -r
netstat -apn|grep nginx
```

### 对nginx服务执行停止/启动/重新读取配置文件操作

```shell script
#启动nginx服务
systemctl start nginx.service
#停止nginx服务
systemctl stop nginx.service
#重启nginx服务
systemctl restart nginx.service
#重新读取nginx配置(这个最常用, 不用停止nginx服务就能使修改的配置生效)
systemctl reload nginx.service
```

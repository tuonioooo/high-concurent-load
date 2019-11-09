# CentOS7 添加nginx系统服务

创建nginx启动命令脚本

```
vi /etc/init.d/nginx
```

插入以下内容, 注意修改PATH和NAME字段, 匹配自己的安装路径

```
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

设置执行权限

```
chmod a+x /etc/init.d/nginx
```

注册成服务

```
chkconfig --add nginx
```

设置开机启动

```
chkconfig nginx on
```

重启, 查看nginx服务是否自动启动

```
shutdown -h 0 -r
netstat -apn|grep nginx
```

对nginx服务执行停止/启动/重新读取配置文件操作

```
#启动nginx服务
systemctl start nginx.service
#停止nginx服务
systemctl stop nginx.service
#重启nginx服务
systemctl restart nginx.service
#重新读取nginx配置(这个最常用, 不用停止nginx服务就能使修改的配置生效)
systemctl reload nginx.service
```




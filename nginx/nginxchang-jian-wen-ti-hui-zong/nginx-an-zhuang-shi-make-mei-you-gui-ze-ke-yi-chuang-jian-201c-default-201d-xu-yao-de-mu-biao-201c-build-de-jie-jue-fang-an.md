# nginx 安装时 make: 没有规则可以创建“default”需要的目标“build” 的解决方案

一般套路安装ing，是用的下面这条语句吧：

./configure --prefix=/usr/local/nginx

结果 make 的时候就蹦出来一个错误：

make: \*\*\* 没有规则可以创建“default”需要的目标“build”

看了 configure 时的提示，安装 zlib 和 pcre，重试，解决。


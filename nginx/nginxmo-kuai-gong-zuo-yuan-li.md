# Nginx模块工作原理



nginx的模块从结构上分为3种：

核心模块：HTTP模块，EVENT模块，MAIL模块。

基础模块：HTTPAccess模块，HTTP FastCGI模块，HTTP Proxy模块，HTTP Rewrite模块。

第三方模块：HTTP Upstream Hash模块，Notice模块，HTTP Access Key模块。

  


从功能上分为3种：

Handles（处理器模块）：直接处理请求，并进行输出内容和修改headers信息，Handles处理器模块一般只能有一个。

Filters（过滤器模块）：对其他处理器模块输出内容进行修改操作。最后由Nginx输出。

Proxies（代理类模块）：与后端一些服务，比如FastCGI进行交互。实现服务代理和负载均衡等功能。

  


Nginx分为2种工作模式。

1.单工作进程模式：除主进程外，还有一个工作进程，且工作进程是单线程的。

2.多工作进程模式：每个工作进程包含多工作线程。

Nginx默认是但工作进程模式。

  


Nginx模块属于静态编译方式，启动Nginx后，会直接加载所有模块。

  


Nginx每个模块都有可能处理某个请求，但同一个处理请求只能由一个模块完成。


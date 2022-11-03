# nginx工作原理

## nginx进程模型

nginx有一个主进程(`master`)和几个工作进程(`worker`)。  

主进程的主要目的是读取和评估配置，并维护工作进程。工作进程会对请求进行实际处理。nginx使用基于事件的模型和依赖于操作系统的机制来有效地在工作进程之间分发请求。工作进程数在配置文件中定义，可以针对给定配置进行修复，也可以自动调整为可用CPU内核数（请参阅[worker\_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)）。

nginx及其模块的工作方式在配置文件中确定。默认情况下，配置文件被命名`nginx.conf`并放在目录`/usr/local/nginx/conf`中`/etc/nginx`，或`/usr/local/etc/nginx`。

启动方式有两种：

1. 单进程启动：此时系统中仅有一个进程，该进程既充当master进程的角色，也充当worker进程的角色。
2. 多进程启动：此时系统有且仅有一个master进程，至少有一个worker进程工作。

## nginx模块工作原理

### nginx的模块从结构上分为3种：

* 核心模块：HTTP模块，EVENT模块，MAIL模块。
* 基础模块：HTTPAccess模块，HTTP FastCGI模块，HTTP Proxy模块，HTTP Rewrite模块。
* 第三方模块：HTTP Upstream Hash模块，Notice模块，HTTP Access Key模块。

### nginx的模块从功能上分为3种：

* Handles（处理器模块）：直接处理请求，并进行输出内容和修改headers信息，Handles处理器模块一般只能有一个。
* Filters（过滤器模块）：对其他处理器模块输出内容进行修改操作。最后由Nginx输出。
* Proxies（代理类模块）：与后端一些服务，比如FastCGI进行交互。实现服务代理和负载均衡等功能。

### Nginx分为2种工作模式。

1.单工作进程模式：除主进程外，还有一个工作进程，且工作进程是单线程的。

2.多工作进程模式：每个工作进程包含多工作线程。

Nginx默认是单工作进程模式。

Nginx模块属于静态编译方式，启动Nginx后，会直接加载所有模块。

Nginx每个模块都有可能处理某个请求，但同一个处理请求只能由一个模块完成。

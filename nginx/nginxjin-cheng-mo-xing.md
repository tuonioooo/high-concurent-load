# Nginx进程模型

### 概述

nginx有一个_**主进程（**_master_**）**_和几个_**工作进程（**_worker_**）**_。主进程的主要目的是读取和评估配置，并维护工作进程。工作进程会对请求进行实际处理。nginx使用基于事件的模型和依赖于操作系统的机制来有效地在工作进程之间分发请求。工作进程数在配置文件中定义，可以针对给定配置进行修复，也可以自动调整为可用CPU内核数（请参阅[worker\_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)）。

nginx及其模块的工作方式在配置文件中确定。默认情况下，配置文件被命名`nginx.conf`并放在目录`/usr/local/nginx/conf`中`/etc/nginx`，或`/usr/local/etc/nginx`。

### 启动方式有两种：

1. 单进程启动：此时系统中仅有一个进程，该进程既充当master进程的角色，也充当worker进程的角色。
2. 多进程启动：此时系统有且仅有一个master进程，至少有一个worker进程工作。




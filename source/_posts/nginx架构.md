---
title: nginx架构
date: 2018-06-25 14:57:01
tags: 
- 架构
- 读书笔记
categories:
- nginx
---

# nginx的进程模型 
nginx在启动后，在unix系统中会以daemon的方式在后台运行，后台进程包含一个master进程和多个worker进程. nginx是以多进程的方式来工作的，当然nginx也是支持多线程的方式的，只是我们主流的方式还是多进程的方式，也是nginx的默认方式

master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程.而基本的网络事件，则是放在worker进程中来处理了.多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的.一个请求，只可能在一个worker进程中处理，一个worker进程，不可能处理其它进程的请求.worker进程的个数是可以设置的，一般我们会设置与机器cpu核数一致

![nginx架构](http://p3euxxfa8.bkt.clouddn.com/2018-06-25-15-00-15.png)

操作nginx只需要与master进程通信即可. 

nginx采用这种进程模型的好处:
- 独立的进程, 不需要加锁, 省掉了锁的开销, 便于定位问题
- 进程间相互不影响, 降低了风险

# nginx的事件处理
nginx采用异步非阻塞来处理请求. 

对于一个基本的web服务器来说，事件通常有三种类型，网络事件、信号、定时器. 网络事件通过异步非阻塞可以很好的解决掉; 对于nginx来说，如果nginx正在等待事件（epoll_wait时），如果程序收到信号，在信号处理函数处理完后，epoll_wait会返回错误，然后程序可再次进入epoll_wait调用; 对于定时器, 由于epoll_wait等函数在调用的时候是可以设置一个超时时间的，所以nginx借助这个超时时间来实现定时器

# nginx基础概念
## connection
connection 是对tcp的封装, 可以很方便的使用nginx来处理与连接相关的事情, nginx中的http请求的处理就是建立在connection上的.

结合一个tcp连接的生命周期，我们看看nginx是如何处理一个连接的.首先，nginx在启动时，会解析配置文件，得到需要监听的端口与ip地址，然后在nginx的master进程里面，先初始化好这个监控的socket(创建socket，设置addrreuse等选项，绑定到指定的ip地址端口，再listen)，然后再fork出多个子进程出来，然后子进程会竞争accept新的连接.此时，客户端就可以向nginx发起连接了.当客户端与服务端通过三次握手建立好一个连接后，nginx的某一个子进程会accept成功，得到这个建立好的连接的socket，然后创建nginx对连接的封装，即ngx_connection_t结构体.接着，设置读写事件处理函数并添加读写事件来与客户端进行数据的交换.最后，nginx或客户端来主动关掉连接，到此，一个连接就寿终正寝了

nginx通过设置worker_connectons来设置每个进程支持的最大连接数.如果该值大于nofile，那么实际的最大连接数是nofile，nginx会有警告. 

nginx通过一个大小为 worker_connections 的连接池来管理连接, 每个 worker 进程都会有一个独立的连接池. 连接池中保存的不是真正的连接 ,而是一个大小为 worker_connections 的ngx_connection_t数组. 并且, 通过一个链表 free_connections 来保存所有空闲的 ngx_connection_t. 

一个nginx能建立的最大连接数, 应该是 worker_connections * worker_processes. 当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是worker_connections * worker_processes，而如果是HTTP作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/2.因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接.

## request
http请求, 在nginx中的结构体为 ngx_http_request_t. nginx_http_request_t 是一个对 http请求的封装, 一个http请求包括请求行, 请求头, 请求体, 响应行, 响应头, 响应体. 

对于nginx来说, 处理一个请求是从 ngx_http_init_request 开始的. 在这个函数中，会设置读事件为ngx_http_process_request_line，也就是说，接下来的网络事件，会由ngx_http_process_request_line来执行.  nginx为提高效率, 采用状态机来解析请求行, 而且对于method的比较, 采用的是将4个字符转换为整型比较. 整个请求行解析到的参数，会保存到ngx_http_request_t结构当中. 

解析完请求行后, nginx会设置读事件的handleer 为 ngx_http_request_headers, 后续的请求就在这里解析与处理. 解析到的请求头会保存到ngx_http_request_t的域headers_in中，headers_in是一个链表结构，保存所有的请求头。而HTTP中有些请求是需要特别处理的，这些请求头与请求处理函数存放在一个映射表里面，即ngx_http_headers_in，在初始化时，会生成一个hash表，当每解析到一个请求头后，就会先在这个hash表中查找，如果有找到，则调用相应的处理函数来处理这个请求头。比如:Host头的处理函数是ngx_http_process_host

当nginx解析到两个回车换行符时，就表示请求头的结束，此时就会调用ngx_http_process_request来处理请求了. ngx_http_process_request会设置当前的连接的读写事件处理函数为ngx_http_request_handler，然后再调用ngx_http_handler来真正开始处理一个完整的http请求. 最终是调用ngx_http_core_run_phases来处理请求，产生的响应头会放在ngx_http_request_t的headers_out中

![请求流程](http://p3euxxfa8.bkt.clouddn.com//18-6-25/60519689.jpg)

## keepalive
http请求是请求应答式的，如果我们能知道每个请求头与响应体的长度，那么我们是可以在一个连接上面执行多个请求的，这就是所谓的长连接，但前提条件是我们先得确定请求头与响应体的长度. 如果请求需要有body, 可以在请求头中指定content_length表明body的大小, 否则返回400错误. 

能否使用长连接，也是有条件限制的。如果客户端的请求头中的connection为close，则表示客户端需要关掉长连接，如果为keep-alive，则客户端需要打开长连接，如果客户端的请求中没有connection这个头，那么根据协议，如果是http1.0，则默认为close，如果是http1.1，则默认为keep-alive。 同时nginx会设置一个最大等待时间, 以防止长连接长期占用连接而无请求传入.

## pipe
## lingering_close
延迟关闭, 当nginx要关闭连接时, 并非立即关闭连接, 而是先关闭tcp连接的写, 再等待一段时间后关闭连接的读. 





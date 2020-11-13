## 反向代理

###反向代理基础原理
1. 概念定义
    + 反向代理服务器介于永和和真实服务器之间，提供请求和响应的中转服务
    + 对于用户而言，访问反向代理服务器就是访问真实服务器
    + 反向代理可以有效降低服务器的负载消耗，提升效率
2. 原理图解
    + ![反向代理](/img/8.png)
3. 反向代理的优势
    + 隐藏真实服务器
    + 便于横向扩充后端动态服务
    + 动静分离，提升系统的健壮性

### 动静分离
1. 概念定义
    + 在web服务器架构中，将动态页面与静态页面或者静态内容接口和动态内容接口分开不同系统访问的架构设计方法，进而提升整个服务访问性能和可维护性
2. web资源分类
    + 静态资源 jpg/css/js/html
    + 动态资源 jsp/asp/php
3. web请求图解
    + ![web请求](/img/9.png)

### nginx作为反向代理所支持的协议
+ ![nginx作为反向代理所支持的协议](/img/10.png)

### upstream模块--用于定义上游服务器
1. 基本功能
    + upstream模块用于定义上游服务器的相关信息
2. 图解原理
    + ![web请求](/img/11.png)
3. 常用指令
    + upstream 段名，以{开始，}结束，中间定义上游服务器URL
    + server 定义上游服务地址
    + zone 定义共享内存，用于夸worker子进程
    + keepalive 对上游服务启用长链接
    + keepalive_requests 一个长连接可以处理最多请求个数
    + keepalive_timeout 空闲情况下，一个长连接的超时时长
    + hash 哈希负载均衡算法
    + ip_hash 依据ip进行哈希计算的负载均衡算法
    + least_conn 最少连接数负载均衡算法
    + least_time 最短响应时间负载均衡算法
    + random 随机负载均衡算法

### upstream模块指令用法
参考proxy.conf
1. upstream 
    + 语法：upstream name {...}
    + 上下文：http
    + 示例：upstream{... ... ...}
2. server
    + 语法：server address [parameters]
    + 上下文：upstream
    + parameters可选值
        1. weight=number 权重值默认为1
        2. max_conns=number 上游服务器的最大并发连接数
        3. fail_timeout=time 服务器不可用的判定时间
        4. max_fails=number 服务器不可用的检查次数
        5. backup 备份服务器，仅当其他服务器都不可用时
        6. down 标记服务器长期不可用，离线维护
3. keepalive
    + 语法：keepalive connections
    + 上下文：upstream
    + 示例：keepalive 16 
4. keepalive_requests
    + 语法：keepalive_requests number
    + 上下文：upstream
    + 示例：keepalive_requests 100
5. keepalive_timeout
    + 语法：keepalive_timeout time
    + 上下文：upstream
    + 示例：keepalive_timeout 60s
6. queue--所有上游服务器不可用时，请求会被放到队列中等待
    + 语法：queue number [timeout=time]
    + 上下文：upstream
    + 示例：queue 100 timeout=30s

### proxy_pass指令
+ 基本功能
    1. 由http_proxy模块提供（ngx_http_proxy_module）
    2. 默认已经被编译进nginx
    3. 禁用必须通过`--without-http_proxy_module`
+ 语法结构
    1. proxy_pass URL
    2. 上下文：location if limit_except
    3. 示例一：proxy_pass http://127.0.0.1:8080
    4. 示例二：proxy_pass http://127.0.0.1:8080/proxy
+ URL参数原则
    1. URL必须以http或https开头
    2. URL中可以携带变量
    3. URL中是否携带uri，会直接影响发往上游请求的URL

### proxy_pass用法常见误区
参考proxy_pass.txt

### 代理场景下nginx接收用户包体的处理方式
+ 处理方式
    1. 接收完全部包体再发送
    2. 一边接收包体一边发送
+ 相关指令
    1. proxy_request_buffering--开启缓存区，缓存请求包体
        + 语法 proxy_request_buffering on | off
        + 默认值 proxy_request_buffering on
        + 上下文 http server location
        + on使用场景 吞吐量要求高 上游服务器并发处理能力低
        + off使用场景 更及时的响应 减少nginx磁盘IO
    2. client_max_body_size--允许的请求体大小
        + 语法 client_max_body_size size
        + 默认值 client_max_body_size 1M
        + 上下文 http server location
    3. client_body_buffer_size--缓冲区大小
        + 语法 client_body_buffer_size size
        + 默认值 client_body_buffer_size 8K|16K
        + 上下文 http server location
    4. client_body_in_single_buffer--请求体尽可能存入缓存
        + 语法 client_body_in_single_buffer on|off
        + 默认值 client_body_in_single_buffer off
        + 上下文 http server location
    5. client_body_temp_path--请求体存储磁盘的目录
        + 语法 client_body_temp_path path [level1] [level2]
        + 默认值 client_body_temp_path client_body_temp
        + 上下文 http server location
    6. client_body_in_file_only--请求体默认存储在磁盘
        + 语法 client_body_in_file_only on|clean|off
        + 默认值 client_body_in_file_only off
        + 上下文 http server location
    7. clent_body_timeout--指定超时时间
        + 语法 clent_body_timeout time
        + 默认值 clent_body_timeout 60s
        + 上下文 http server location

### 代理场景下nginx更改发往上游的请求
+ 请求行信息修改指令
    1. proxy_method--修改请求方法
        + 语法 proxy_method method
        + 默认值 -
        + 上下文 http server location
    2. proxy_http_version--修改请求协议
        + 语法 proxy_http_version 1.0/1.1
        + 默认值 proxy_http_version 1.0
        + 上下文 http server location
+ 请求头部修改指令
    1. proxy_set_header--修改请求头部
        + 语法 proxy_set_header field value
        + 默认值 proxy_set_header Host $proxy_host
        + 上下文 http server location
    2. proxy_pass_request_header--客户端请求头信息是否发往上游
        + 语法 proxy_pass_request_header on|of
        + 默认值 proxy_pass_request_header on
        + 上下文 http server location
+ 请求包体修改指令
    1. proxy_set_body--修改请求包体
        + 语法 proxy_set_body value
        + 默认值 -
        + 上下文 http server location
    2. proxy_pass_request_body--客户端请求包体是否发往上游
        + 语法 proxy_pass_request_body on|off
        + 默认值 proxy_pass_request_body on
        + 上下文 http server location

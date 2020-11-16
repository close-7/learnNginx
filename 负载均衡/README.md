## 负载均衡
+ 概念定义
    1. 将请求代理到多态服务器去执行，就称之为负载均衡
    2. 这里的多台服务器通常承担一样的功能任务

## 配置实现Nginx对上游服务负载均衡
```
upstream back_end{
    #加权轮询-设置权重weight
    server 127.0.0.1:8080 weight=3 max_conns=1000 fail_timeout=10s max_fails=2;
    server 127.0.0.1:8081 weight=2 max_conns=1000 fail_timeout=10s max_fails=2;
    keepalive 32;
    keepalive_requests 50;
    keepalive_timeout 30s;
}
```

### upstream指令 --配置上游服务段信息
+ 语法: upstream name {...}
+ 默认值： -
+ 上下文：http
+ 示例： upstream {... ...}

### 负载均衡算法-哈希算法
+ 概念定义：
    1. 哈希算法是将任意长度的二进制值映射为较短的固定长度的二进制值（串），这个小的二进制值我们称之为哈希值
    2. 散落明文到哈希值的映射是不可逆的
+ 常用指令
    1. hash指令
        + 语法：hash key [consistent]
        + 默认值：-
        + 上下文：upstream
+ 示例
```
upstream back_end{
    #哈希算法--将根据变量request_uri的值进行分配
    #request_uri哈希值不变，指向的服务器就不会变(方便登录缓存，因为服务器切换的登录失效问题)
    hash $request_uri;
    server 127.0.0.1:8080 ;
    server 127.0.0.1:8081 ;
    keepalive 32;
    keepalive_requests 50;
    keepalive_timeout 30s;
}
```

### 负载均衡算法-ip_hash算法
根据客户端ip地址哈希运算，只要客户端地址不变，分配的应用服务器地址不变
```
upstream back_end{
    ip_hash;
    server 127.0.0.1:8080 ;
    server 127.0.0.1:8081 ;
    keepalive 32;
    keepalive_requests 50;
    keepalive_timeout 30s;
}
```

### 负载均衡算法-最少连接数算法
+ 概念定义
    1. 从上游服务器，挑选一台当前已建立连接数最少的去分配请求
    2. 极端情形下退化为轮询算法
+ 常用指令
    1. least_conn指令--最少连接
        + 概念定义 模块ngx_http_upstream_least_conn_module
        + 禁用通过--without-http_upstream_least_conn_module
        + 语法：least_conn
        + 默认值：-
        + 上下文： upstream
    2. zone指令--开辟内存空间
        + 语法： zone name [size]
        + 上下文： upstream
+ 示例
```
upstream back_end{
    zone test 10M;
    least_conn;
    server 127.0.0.1:8080 ;
    server 127.0.0.1:8081 ;
    keepalive 32;
    keepalive_requests 50;
    keepalive_timeout 30s;
}
```

## Nginx针对上游服务返回异常时的容错机制
+ proxy_next_upstream [error|...]针对上游服务返回异常进行分发
    + 指令： proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 |
    http_504 | http_403 | http_404 | http_429 | non_idempotent | off
    + 默认值：proxy_next_upstream error | timeout;
    + 上下文：http server location
    + 指令参数解析
        1. error 向上游服务器传输请求或读取响应头发生错误
        2. timeout 向上游服务器传输请求或读取响应头发生超时
        3. ivalid_header 上游服务器返回无效的响应
        4.  http_500|... HTTP响应码为500|...时 
        5. off 禁用请求失败转发功能
        6. non_idempotent 非幂等请求失败时是否需要转发下一台上游服务器
+ proxy_next_upstream_timeout 指令---异常时等待多久
    + 语法：proxy_next_upstream_timeout times
    + 默认值：proxy_next_upstream_timeout 0；
    + 上下文：http server location
+ proxy_next_upstream_tries 指令---异常时转发多少次
    + 语法：proxy_next_upstream_tries number
    + 默认值：proxy_next_upstream_tries 0；
    + 上下文：http server location
+ proxy_intercept_errors 指令--上游返回响应码大于300时，是直接将上游响应返回客户端还是按照error_page处理
    + 语法：proxy_intercept_errors on | off
    + 默认值：proxy_intercept_errors on
    + 上下文：http server location
+ 示例：
```
应用服务器
server{
    listen 4040;
    location / {
        return 200 'Return Result For Server 4040\n';
    }
}

server{
    listen 4050;
    location / {
        return 200 'Return Result For Server 4050\n';
    }
}
nginx反向代理服务器
upstream test_server{
    server 192.168.1.1:4040;
    server 192.168.1.1:4050;
}
server {
    listen 80;
    location /test/ {
        proxy_pass http://test_server;
        proxy_next_upstream_timeout 5s;
        proxy_next_upstream_tries 2;
         proxy_next_upstream error;
         error_page 503 /503.html
         proxy_intercept_errors on；
    }
}
```
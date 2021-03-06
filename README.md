# learnNginx
nginx学习

## Nginx 定义
Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。  
Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强。

### Nginx 优势
+ 高并发，高性能
+ 扩展性好（模块化设计）
+ 异步非阻塞的事件驱动模型
+ 高可靠性

### Nginx 安装
+ yum install epel-release -y
+ yum list all | grep nginx
+ yum install nginx -y
+ rpm -ql nginx 查看文件

### Nginx 进程结构
多进程![avatar](/img/1.png)

### linux 信号量
+ kill -l 命令显示
+ 常用

### 使用信号量管理master和worker
+ Master进程
    1. 监控worker进程 CHLD
    2. 管理worker进程
    3. 接收信号 TERM,INT QUIT HUP USR1 USR2 WINCH
+ Worker进程
+ 命令行  

ps -ef | grep nginx 查看nginx启动了几个进程


### nginx配置文件重载的原理
+ reload重载配置文件流程
    1. 向master进程发送HUP信号（reload命令）
    2. master进程检查配置语法是否正确
    3. master进程打开新的监听端口
    4. master进程使用新的配置文件启动新的worker子进程
    5. master进程向老的worker子进程发送QUIT信号
    6. 旧的worker进程关闭检查句柄，处理完当前连接后关闭进程


### nginx的热部署流程
1. 将旧的nginx文件替换成新的nginx文件
2. 向master进程发送USR2信号
3. master进程修改pid文件，加后缀.oldbin
4. master进程用新nginx文件启动新master进程
5. 向旧的master进程WINCH信号，旧的worker子进程退出
![avatar](/img/2.png)

### nginx的模块化管理机制
nginx模块结构图![nginx模块结构图](/img/3.png)

### nginx的配置文件结构
nginx的配置文件结构![配置文件结构](/img/4.png)

### 配置文件main段核心参数用法
参考lesson1
+ 全局段核心参数
+ user USERNAME [GROUP]
    1. 解释：指定运行nginx的worker子进程的属主和属组，其中属组可以不指定
    2. 实例 `user nginx nginx`
+ pid DIR
    1. 解释：指定运行nginx的master主进程的pid文件存放路径
    2. 示例 `pid /opt/nginx/logs/nginx.pid;`
+ worker_rlimit_nofile number
    1. 解释：指定worker子进程可以打开的最大文件句柄数
    2. 示例：`worker_rlimit_nofile:20480;`
+ worker_rlimit_core size
    1. 解释：指定worker子进程异常终止后的core文件，用于记录分析问题
    2. 示例：`worker_rlimit_core 50M;` `working_directory /opt/nginx/tmp;`
+ worker_processes number|auto
    1. 解释：指定nginx启动的worker子进程数量
    2. 示例：`worker_processes 4`  `worker_prcesses auto`
+ worker_cpu_affinity cpumask1 cpumask2...
    1. 解释：将每个worker子进程与我们的cpu无力核心绑定
    2. 示例： 
    ```
    worker_cpu_affinity 0001 0010 0100 1000;四个物理核心，四个worker子进程
    worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000；八个物理核心，八个worker子进程
    worker_cpu_affinity 01 10 01 10；两个物理核心，四个worker子进程
    ```
    备注：将每个worker子进程与特定cpu物理核心绑定，优势在于：避免同一个worker子进程在不同的cpu核心上切换，缓存失效，降低性能。其并不能真正的避免进程切换。
+ worker_priority number
    1. 解释：指定worker子进程的nice值，以调整运行nginx的优先级，通常设定为负值，以优先调用nginx
    2. 示例：`worker_priority -10`;
    3. 备注：linux默认进程的优先级值是120，值越小越优先；nice设定范围为-20到+19
+ worker_shutdown_timeout time
    1. 解释：指定worker子进程优雅退出时的超时时间
    2. 示例：`worker_shutdown_timeout 5s`
+ timer_resolution intcrval
    1. 解释：worker子进程内部使用的计时器精度，调整时间间隔越大，系统调用越少，有利于性能提升；反之，系统调用越多，性能下降
    2. 示例 `timer_resolution 100ms`
+ daemon on|off
    1. 解释：设定nginx的运行方式，前台还是后台，前台用于调试，后台用于生产
    2. 示例：`daemon on`

### 配置文件events段核心参数用法
参考lesson1
+ use-->nginx使用何种事件驱动模型
    1. 语法：use method
    2. method可选值: select,poll,kqueue,epoll,/dev/poll,eventport
    3. 默认配置：无
    4. 推荐配置：不指定，让nginx自己选择
+ worker_connections-->worker子进程能够处理的最大并发连接数
    1. 语法：worker_connections 1024
    2. 推荐配置：worker_connections 65535/worker_procsses|65535
+ accept_mutex-->是否打开负载均衡互斥锁
    1. 语法：accept_mutex on|off
    2. 可选值：on|off
    3. 默认配置：accept_mutex off；
    4. 推荐配置： accept_mutex on；
+ accept_nutex_delay-->新连接分配给worker子进程的超时时间
    1. 语法：accept_mutex_delay time
    2. 默认配置：accept_mutex_delay 500ms
    3. 推荐配置：accept_mutex_delay 200ms
+ lock_file-->负载均衡互斥锁文件存放路径
    1. 语法：lock_file file
    2. 默认配置 lock_file logs/nginx.lock
+ muti_accept-->worker子进程可以接收的新连接个数
    1. 语法：muti_accept on|off
    2. 可选值：on|off
    3. 默认配置：muti_accept off；
    4. 推荐配置： muti_accept on； 



### 配置文件server_name指令用法
参考lesson1
+ 语法结构：server_name name1 name2 name3...;
    1. 示例: server_name www.nginx.com 精确匹配
    2. 示例: server_name *.nginx.org
    3. 示例: server_naem ~^www\.imooc\.*$

### server_name指令优先级
+ 精确匹配最高
+ 左侧通配符匹配
+ 右侧通配符匹配
+ 正则匹配最低

### root和alias指令用法区别
参考文件夹root&alias
+ 语法结构：
    1. root：
        + 语法：root path；
        + 上下文：http server location if
    2. alias：
        + 语法 alias path
        + 上下文：location

+ 相同点：URI到磁盘文件的映射
+ 区别：root会将定义路径与URI叠加；alias则只取定义路径
+ 示例1：
```
location /picture{
    root /opt/nginx/html/picture;
}
客户端请求www.test.com/picture/1.jpg，则对应磁盘映射
路径/opt/nginx/html/picture/picture/1.jpg
```
+ 示例2：
```
location /picture{
    alias /opt/nginx/html/picture;
}
客户端请求www.test.com/picture/1.jpg，则对应磁盘映射
路径/opt/nginx/html/picture/1.jpg
```
+ 注意 使用alias时，末尾一定要加/;alias只能位于location中


### location的基础用法
参考文件夹location
+ 语法结构：location [=|~|~*|^~] uri {...}
+ 上下文：server location
+ 匹配规则及含义：
    1. = 精准匹配：location=/images/{...}
    2. ~ 正则匹配区分大小写：location ~ \.(jpg|gif)${...}
    3. ~* 正则匹配不区分大小写：location ~* \.(jpg|gif)${...}
    4. ^~ 匹配到即停止搜索：location ^~/images/{...}
    5. 不带任何符号：location / {...}

### location规则的匹配顺序
+ =
+ ^~
+ ~
+ ~*
+ 不带任何字符

### location中URL结尾的反斜线
+ 不带/:location /test{}   -->找test文件夹，如果没有则视为test文件查找
+ 带/:location /test/{} --》找test文件夹，没有则返回404

### stub_status模块的用法
+ 指令： stub_status;
+ 低于1.5.7：stub_status on;
+ 上下文：server location;
+ 示例：`location /uri {stub_status}`
+ 效果示例
```
Active connections:2
server accepts handled requests
883 883 928
Reading: 0 Writing: 1 Waiting:1
```
+ 状态项及含义
    1. Active connections：活跃的链接数量
    2. accepts 接受的客户端连接总数
    3. handled 处理的客户端连接总数
    4. requests 客户端总的请求数量
    5. Reading 读取客户端的连接数
    6. Writing 响应数据到客户端的连接数
    7. Waiting 空闲客户端请求连接数量

### connection和request
1. connection是连接，及常说的tcp连接，三次握手，状态机
2. request是请求，例如http请求，无状态的协议
3. request是必须建立在connection之上
4. 图片示例![avatar](/img/5.png)

### limit_conn模块-connection链接限制模块
参考limit_conn文件夹
+ 基本功能
    1. 用于限制客户端默认连接数
    2. 默认编译进nginx，通过`--without-http_limit_conn_module`禁用
    3. 使用共享内存，对所有worker子进程生效
+ 常用指令
    1. limit_conn_zone-定义共享内存空间
        + 语法：limit_conn_zone key zone = name.size
        + 上下文：http
        + 示例 `limit_conn_zone $binary_remote_addr zone=limit_addr:10m;`
    2. limit_conn_status-定义限制行为发生时返回给客户端的状态
        + 语法：limit_conn_status code
        + 默认值：limit_conn_status 503
        + 上下文：http server location
        + 示例：`limit_conn_status 503;`
    3. limit_conn_log_level-定义限制行为发生时日子记录等级
        + 语法：limit_conn_log_level info|notice|warn|error
        + 默认值：limit_conn_log_level warn;
        + 上下文：http server location 
    4. limit_conn-真正定义限制客户端并发连接数
        + 语法：limit_conn zone number
        + 上下文：http server location

### limit_req模块-request请求限制模块
参考limit_req文件夹
+ 基本功能
    1. 用于限制客户端处理请求的平均速率
    2. 默认编译进nginx，通过`--without-http_limit_req_module`禁用
    3. 使用共享内存，对所有worker子进程生效
    4. 限流算法： leaky_bucket ![leaky_bucket](/img/6.png)
+ 常用指令
    1. limit_req_zone-定义共享内存空间
        + 语法：limit_req_zone key zone = name.size rate=rate
        + 上下文：http
        + 示例 `limit_req_zone  $binary_remote_addr zone=limit_req:15m rate=12r/m;` 
    2. limit_req_status-定义限制行为发生时返回给客户端的状态
        + 语法：limit_req_status code
        + 默认值：limit_req_status 503
        + 上下文：http server location
        + 示例：`limit_req_status 503;` 
    3. limit_req_log_level-定义限制行为发生时日子记录等级
        + 语法：limit_req_log_level info|notice|warn|error
        + 默认值：limit_req_log_level error;
        + 上下文：http server location 
    4. limit_req-真正定义限制客户端并发连接数
        + 语法：limit_req zone=name [burst=number] [nodelay|delay=number]
        + 上下文：http server location
        + 示例：`limit_req zone=limit_req;`

### access模块-限制特定ip或网段访问
参考access文件夹
+ allow指令--放行
    1. 语法结构：allow address | CIDR | UNIX | all;
    2. 上下文：http server location limit_except
    3. 示例1：`allow 192.168.0.10`
+ deny指令--阻止
    1. 语法结构：deny address | CIDR | UNIX | all;
    2. 上下文：http server location limit_except
    3. 示例1：`deny 192.168.0.0/24`
+ 组合示例
```
location / {
    deny 192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny all;
}
```

### auth_basic模块-限制特定用户访问
参考auth_basic文件夹
+ 基本功能
    1. 基于HTTP Basic Authentication协议进行用户名密码认证
    2. 默认编译进nginx，通过`--without-http_auth_basic_module`禁用 
+ 常用指令
    1. auth_basic--定义是否开启用户名密码验证功能
        + 语法：auth_basic string|off
        + 默认值：auth_basic off;
        + 上下文：http server location limitz_except
    2. auth_basic_user_file--定义是存储用户名密码的文件
        + 语法：auth_basic_user_file file
        + 默认值：-
        + 上下文：http server location limitz_except
+ 生成密码文件工具
    1. 可执行程序 htpasswd
    2. 所属软件包 httpd-tools
    3. 生成密码文件 `htpasswd -bc encrypt_pass jack 12345`
    4. 添加新用户密码 `htpasswd -b encrypt_pass mike 12345`

### auth_request模块-基于HTTP响应状态码做权限控制
参考auth_request文件夹
+ 功能及原理![auth_request](/img/7.png)
+ 启用禁用：默认未编译进nginx，通过`--with-http_auth_request_module`启用
+ 常用指令
    1. auth_request --定义是否开启用模块
        + 语法：auth_request uri|off
        + 默认值：auth_request off
        + 上下文：http server location 
    2. auth_basic_set--定义是存储用户名密码的文件
        + 语法：auth_basic_set $variable value
        + 默认值：-
        + 上下文：http server location 


### rewrite模块--实现对url重写
参考rewrite文件夹
#### rewrite模块中的return指令
参考rewrite文件夹中return.conf
+ return功能
    1. 停止处理请求，直接返回响应码或者重定向到其他URL
    2. 执行return指令后，location中后续指令将不会执行
    3. 示例 `location / {.... return 404; ....}`

+ return语法结构
    1. 语法： return code[text];return code URL;return URL;
    2. 默认值： -
    3. 上下文：server location if

+ HTTP状态码
    1. 1xx：消息类
    2. 2xx：成功
    3. 3xx：重定向
        + 301:永久重定向
        + 302：临时重定向
    4. 4xx: 客户端错误
    5. 5xx：服务器错误

#### rewrite模块中的rewrite指令
参考rewrite文件夹中rewrite.conf
+ rewrite功能
    1. 根据指定正则表达式匹配规则，重写URL

+ rewrite语法结构
    1. 语法： rewrite regex replacement [flag];
    2. 默认值： -
    3. 上下文：server location if
    4. 示例：`rewrite /images/(.*\.jpg)$ /pic/$1`;

+ flag可选值及含义
    1. last 重写后的URL发起新请求，再此进入server段，重试location中的匹配
    2. break 直接使用重写后的URL，不再匹配其他location中语句
    3. redirect 返回302临时重定向
    4. permanent 返回301永久重定向

+ rewrite执行顺序


#### rewrite模块中的if指令
参考rewrite文件夹中if.conf
+ if 功能
    1. 对某些条件进行判断，根据变量值不同，对url进行不同处理
+ if 语法结构
    1. 语法： if (condition) {}
    2. 默认值： -
    3. 上下文：server location 
    4. 示例：`if($http_user_agent~Chrome){rewrite /(.*)/browser/$1 break;}`;

+ condition用法
    1. $variable 仅为变量时，值为空或以0开头字符串都会被当做false处理
    2. = | != 相等或不等比较
    3. ~ | !~ 正则匹配或非正则匹配
    4. ~* 正则匹配，不区分大小写
    5. -f | !-f 检查文件存在或不存在
    6. -d | !-d 检查目录存在或不存在
    7. -e | !-e 检查文件，目录，符号链接等存在或不存在
    8. -x | !-x 检查文件可执行或不可执行


### autoindex模块用法
参考autoindex文件夹中
+ 基本功能
    1. 用户请求以/结尾时，列出目录结构
+ 常用指令
    1. autoindex--是否开启列出
        + 语法：autoindex on|off 
        + 默认值：autoindex off
        + 上下文：http server location
    2. autoindex_exact_size--是否显示文件大小（字节）
        + 语法：autoindex_exact_size on|off 
        + 默认值：autoindex_exact_size on
        + 上下文：http server location
    3. autoindex_format--返回目录结构以哪种形式
        + 语法：autoindex_format html|xml|json|jsonp
        + 默认值：autoindex_format html
        + 上下文：http server location
    4. autoindex_localtime--返回目录结构时间格式
        + 语法：autoindex_localtime on|off 
        + 默认值：autoindex_localtime off
        + 上下文：http server location


### Nginx变量分类
参考 var文件夹
+ TCP连接产生的变量
    1. remote_addr 客户端IP地址
    2. remote_port 客户端端口
    3. server_addr 服务端IP地址
    4. server_port 服务端端口
    5. server_protocol 服务端协议
    6. connection TCP连接的序号，递增
	7. connection_request TCP连接当前的请求数量，对于keepalive连接有意义
	8. proxy_protocol_addr 如果使用proxy_protocol协议，则返回原始用户的地址，否则为空
	9. proxy_protocol_port 如果使用proxy_protocol协议，则返回原始用户的端口，否则为空
    10. binary_remote_addr 二进制格式的客户端IP地址
+ HTTP请求过程中的常用变量
    1. uri 请求的url，不包含参数
    2. request_uri 请求的url，包含参数
    3. scheme 协议名，http或https
    4. request_method 请求方法
    5. request_length 全部请求的长度，包括请求行，请求头，和请求体
    6. args 全部参数字符串
    7. arg_参数名 特定参数值
    8. is_args URL中有参数则返回？；否则返回空
    9. query_string 与args相同
    10. remote_user 由HTTP Basic Authentiction协议传入的用户名
+ HTTP请求过程中的特殊变量
    1. host 先看请求行，再看请求头，最后找server_name
    2. http_user_agent 用户浏览器
    3. http_referer 从哪些链接过来的请求
    4. http_via 经过一层代理服务器，添加对应代理服务器的信息
    5. http_x_forwarded_for 获取用户真实ip
    6. http_cookie 用户cookie
+ Nginx处理HTTP请求产生的变量
    1. request_time 处理请求已消耗的时间
    2. request_completion 请求处理完成返回ok，否则返回空
    3. server_name 匹配上请求的server_name值
    4. https 若开启https，则返回on，否则返回空
    5. request_filename 磁盘文件系统待访问文件的完整路径
    6. document_root 由uri和root/alias规则生成的文件夹路径
    7. realpath_root 讲document_root中的软连接换成真实路径
    8. limit_rate 返回响应时的速度上限值
+ Nginx返回响应的变量
+ Nginx内部运行时的变量






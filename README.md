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
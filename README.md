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
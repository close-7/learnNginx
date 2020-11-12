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
多进程![avatar](//1.png)

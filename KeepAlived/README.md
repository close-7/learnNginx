## Nginx高可用继承
传统c/s模式客户服务器模式

### 服务可用三要素
+ IP地址
+ 监听端口（软件程序启动）
+ 数据文件

### VRRP协议原理--IP地址在多台服务器间转移
+ 核心概念：
    1. 虚拟网关--一个Master网关，多个Backup网关
    2. Master网关--实际承载报文转发的节点，主节点
    3. Backup网关--主节点故障后转移节点，备用节点
    4. 虚拟ip地址--虚拟网关对外提供服务的IP地址
    5. ip拥有者地址--真实提供服务的节点，通常为主节点
    6. 虚拟mac地址--回应ARP请求是使用的虚拟MAC地址
    7. 优先级
    8. 非抢占式
    9. 抢占式

### KeepAlived 软件架构
+ 适用场景
    1. 高可用LVS（虚拟IP转移，生成ipvs规则，RS健康状态监测）
    2. 高可用其他服务（虚拟IP转移，编写脚本实现服务启动/停止）

+ 核心组件
    1. vrrp stack vrrp协议的实现
    2. ipvs wrapper 为集群内的节点生成ipvs规则
    3. checkers 对集群内所有的RS做健康状态检测
    4. 控制组件 配置文件解析和加载

### 使用KeepAlived 配置实现ip在多服务器节点上漂移
+ 安装 `yum install keepalived`
+ 查看配置 `vim /etc/keepalived/keepalived.conf`

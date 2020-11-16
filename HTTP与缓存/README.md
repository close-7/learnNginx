## 缓存基础
+ 客户端缓存
    1. 优势： 直接在本地获取内容，没有网络消耗，响应最快
    2. 缺点： 仅对单一用户生效
+ 服务器缓存
    1. 优势： 对所有用户生效，有效的降低上游应用服务器的压力
    2. 缺点： 用户任然有网络消耗 

### 缓存相关指令
+ proxy_cache指令--对上游应用服务器返回内容缓存
    + 语法：proxy_cache zone | off;
    + 默认值：proxy_cache off;
    + 上下文：http server location

+ proxy_cache_path指令--指定缓存内容磁盘位置和名称，空间大小等等
    + 语法：proxy_cache_path path [level=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size][manager_files=number] [manager_sleep=time][manager_threshold=time][loader_files=number] [loader_sleep=time][loader_threshold=time][purger=on|off][purger_files=number][purger_sleep=time][purger_threshold=time];
    + 可选参数含义
        1. path：缓存文件的存放路径
        2. level：path的目录层级
        3. user_temp_path：off直接使用path路径，on使用proxy_temp_path路径
        4. keys_zone：name是共享内存名，size是共享内存大小
        5. inactive：在指定时间内没有被访问缓存会被清理；默认10分钟
        6. max_size：设定最大缓存文件大小，超过将由CM清理
        7. manager_files：CM清理一次缓存文件，最大清理文件数；默认100；
        8. manager_sleep：CM清理后进程的休眠时间，默认200毫秒
        9. manager_threshold：CM清理一次最长耗时；默认50毫秒
        10. loade_files：CL载入文件到共享内存，每批最多文件数；默认100；
        11. loader_sleep：CL加载缓存文件到内存后，进程休眠时间，默认200毫秒
        12. loader_threshold：CL每次载入文件到共享内存的最大消耗；默认50毫秒
    + 默认值：proxy_cache_path off;
    + 上下文：http 

+ proxy_cache_key指令--指定缓存内容保存的key信息
    + 语法：proxy_cache_key string
    + 默认值：proxy_cache_key $scheme$proxy_host$request_uri
    + 上下文：http server location

+ proxy_cache_valid指令--指定对那些响应码内容缓存
    + 语法：proxy_cache_valid [code...] time
    + 默认值：
    + 上下文：http server location
    + 示例：proxy_cache_valid 60m; 只对200,301,302响应码缓存

+ upstream_cache_status变量（上游应用服务器缓存状态）--帮助看缓存是否命中
    + MISS:未命中缓存
    + HIT:命中缓存
    + EXPIRED:缓存过期
    + STALE:命中了陈旧缓存
    + REVALIDDATED:Nginx验证陈旧缓存依然有效
    + UPDATING: 内容陈旧，但正在更新
    + BYPASS:响应从原始服务器获取

### 缓存用法配置实例
参考cache.conf

### nginx不缓存特定内容
+ proxy_no_cache指令--对特定内容不缓存，string为变量名
    + 语法：proxy_no_cache string
    + 默认值：-
    + 上下文：http server location

+ proxy_cache_bypass指令--对特定请求内容不使用缓存，直接请求上游服务器
    + 语法：proxy_cache_bypass string
    + 默认值：-
    + 上下文：http server location
参考no_cache.conf

### 缓存失效降低上游压力机制
缓存失效，请求穿透nginx导致上游应用服务器压力大
+ 合并源请求机制 参考cache.conf
    + proxy_cache_lock指令-将请求同一资源的请求合并，只发一个请求到上游，缓存后，所有请求从nginx缓存中返回
        1. 语法：proxy_cache_lock on|off
        2. 默认值：proxy_cache_lock off
        3. 上下文：http,server,location
    + proxy_cache_lock_timeout指令-等待缓存完成时间，超时则放开已合并请求（发送所有剩余请求）
        1. 语法：proxy_cache_lock_timeout time
        2. 默认值：proxy_cache_lock_timeout 5s
        3. 上下文：http,server,location
    + proxy_cache_lock_age指令-等待缓存完成时间，超时则再发送一个请求（一个一个发送所有剩余请求）
        1. 语法：proxy_cache_lock_age time
        2. 默认值：proxy_cache_lock_age 5s
        3. 上下文：http,server,location
+ 启用陈旧缓存
    + proxy_cache_use_stale指令-特定条件下启用nginx陈旧缓存
        1. 语法：proxy_cache_use_stale error|timeout|invalid_header|updating|off|http_500...
        2. 默认值：proxy_cache_use_stale off
        3. 上下文：http,server,location
    + proxy_cache_background_update指令-nginx自身向上游服务器发送请求，缓存。
        1. 语法：proxy_cache_background_update on | off
        2. 默认值：proxy_cache_background_update off
        3. 上下文：http,server,location

### 第三方缓存清除模块ngx_cache_purge
+ 模块功能
    1. 根据接收到的http请求立即清除缓存
    2. 使用--add-module指令添加到nginx中
+ 常用指令
    1. proxy_cache_gurge指令--指定清除清除哪个内存里的哪个缓存
        + 语法 proxy_cache_gurge zone_name key
        + 默认值 -
        + 上下文 http server location

### CM缓存管理进程  CL缓存加载进程




## HTTP
### HTTP协议存在的问题
+ 数据使用明文传输，可能被黑客窃取
+ 报文的完整性无法验证，可能被黑客篡改
+ 无法验证通信双发的身份，可能被黑客伪装
+ 网络模型
    1. 应用层（http）--》封装传递给传输层时，是明文状态
    2. 传输层（TCP/UDP)
    3. 网络层（IP）
    4. 数据链路层


### HTTPS
+ 所谓https，其实只是身披TLS/SSL协议外壳的http
+ https并非一个应用层协议
+ 网络模型
    1. 应用层（http）
    2. TLS/SSL协议
    3. 传输层（TCP/UDP)
    4. 网络层（IP）
    5. 数据链路层
+ 明文传输--加密算法；完整性问题--数字签名；身份认证--数字证书

### 加密算法
+ 对称加密 DES AES 3DES--客户端服务端秘钥相同
    1. 优势:解密效率高
    2. 劣势：密钥无法安全传输；密钥数目难于管理；无法提供信息完整性校验
+ 非对称加密 RSA DSA ECC--服务器生成公钥私钥，传输只携带公钥
    1. 优势:服务器只维持一个私钥
    2. 劣势：公钥公开；解密耗时；公钥不包含服务器信息，存在中间人攻击可能性。

### https加密原理
混合使用对称加密和非对称加密
1. 链接建立阶段使用非对称加密
2. 内容传输阶段使用对称加密
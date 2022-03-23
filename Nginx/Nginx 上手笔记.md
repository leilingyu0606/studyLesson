## Nginx 上手笔记

> 概念和功能

为了解决线上并发量大 连接多个服务器的问题

**高性能**（响应 和 并发）的HTTP和**反向代理**web服务器

提供IMAP POP3 SMTP服务协议

**代理**

正向代理：VPN 代理客户端去请求外部服务器

反向代理：代理的是服务器 向外（客户端）的始终只有一个接口

**负载均衡**

通过加权轮询、iphash(还是用redis做session共享)，将更多的请求发送至性能（如内存）更高的服务器上

**动静分离**

nginx可以部署静态资源服务器

**闲话 ** 架构：没有什么是加一层解决不了的事情

> 安装和使用

常用文件 nginx.conf

server { listen 80 ....}

说明监听localhost的默认80端口

> 常用命令

 ./nginx -s stop 停止服务

./nginx -s quit 安全退出

./nginx -s reload （修改 配置文件nginx.conf 后需要）重载

（修改端口后注意需要打开防火墙）

ps aux/grep nginx 查看nginx进程

### nginx实战

> nginx.config文件 

头部 全局配置

events 连接配置

http http配置 + upstream配置（负载均衡配置） + server配置（location配置，含proxy配置）
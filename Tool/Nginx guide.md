## 简介
Nginx (engine x) 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。
## 诞生背景
Nginx是Igor Sysoev[伊戈尔·赛索耶夫] 2002年为俄罗斯访问量第二的Rambler.ru站点开发的一个HTTP站点服务器，第一个公开版本0.1.0发布于2004年10月4日，它被声称每天能够处理5亿个请求，能够运行在几乎所有的主流操作系统上。
Nginx诞生年代，软件界大家讨论激烈的问题就是**C10K**，早期Web服务器如果并发访问过万服务就会自动崩溃。后来的Apache推出V2.4解决并发问题，同时也诞生出来其他服务器解决该问题，Lighttpd,Nginx
## 特点
 -   支持Linux epoll;【轻松支持2/3万并发连接，epoll模式优势，CPU占用率低】
 -   底层实现纯C;【高性能】
 -   核心代码全部用事件触发机制完成
 -   跨平台
 -  开源且社区活跃度高
## 安装
Windows OS下载解压在指定目录即可，[下载地址](https://nginx.org/en/download.html)

Ubuntu安装 sudo apt-get install nginx
安装成功启动Nginx后，可以通过浏览器访问http://localhost 看到Nginx的欢迎页面，Niginx默认端口为80，如果80端口已经被占用，需要修改conf/nginx.conf文件的80端口为可用端口
## 常用命令
 -   启动Nginx, 成功后会开启两个进程，当然何以生成Windows服务实现开机启动`start nginx`
 - 重新加载配置`nginx -s reload`
 - 验证配置是否有效`nginx -t`
 - 停止服务`nginx -s stop` `nginx -s quit`
 - Ubuntu check Nginx status`sudo systemctl status nginx`
 - Ubuntu start Nginx Server`sudo systemctl start nginx`
## 常用功能
 - **静态服务器**
 下面为部署sample站点
```
server {
  listen 7978;
  server_name 127.0.0.1;
  root dist;
  index index.html;
  location / {
	add_header X-Proxy-server $server_addr; //增加Response Header
	error_page 404 = /index.html;           //404调整默认页面
  }
}
```
 - **反向代理**
 ```
 server {
  listen 7778;
  location / {
	proxy_pass http://baidu.com;
  }
}
 ```
 - **负载均衡**
 1) RR(Round Robin) [默认模式]
每个请求按照时间顺序逐一肥培到不同的后端服务器，可以结合max_fails、fail_timeout、weight等综合评估
2) weight(权重)
指定轮询概率，weight和访问比率成正比，用于后端服务器性能不均情况
3) ip_hash
每个请求访问IP的Hash结果分配，使得每个客户端访问一个后端服务器，解决负载集群Session同步问题[hash具有结果一致性]。
4) fair(第三方)
5) (url_hash)
按访问URL的Hash结果来分配请求，使每个URL定向到同一个后端服务器，后端服务器为缓存时比较有效

下面为RR策略配置，访问http://localhost:6606能够循环访问Server1和Server2
```
http {
    ...
    #用upstream指令定义一组反向代理/负载均衡后端服务器池
    upstream test_svr {
      server localhost:6607;
      server localhost:6608;
    }
    server {
      listen 6606;
      location / {
        proxy_pass http://test_svr;#用于指向反向代理的服务器池
      }
    }
}
```
下面为带权RR，访问http://localhost:6606能够循环访问Server1 两次和Server2一次。
```
http {
    ...
    upstream test_svr {
      #ip_hash;
      server localhost:6607 weight=2 max_fails=2 fail_timeout=30s;
      server localhost:6608 weight=1 max_fails=2 fail_timeout=30s;
    }
    server {
      listen 6606;
      location / {
        proxy_pass http://test_svr;
        proxy_set_header Host  $host;#反向代理发送Header
        proxy_set_header X-Forwarded-For  $remote_addr;
      }
    }
}
```

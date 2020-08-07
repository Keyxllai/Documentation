## pm2简介
pm2(process manager)是一个进程管理工具，维护一个进程列表，可以用来管理node进程，并查看node进程的状态，也支持性能监控，负载均衡等功能
## pm2管理node优势
> 1.监听文件编号自动重启  
> 2.支持性能监控  
> 3.负载均衡  

## 安装
```npm install pm2 -g```

## 常用命令
> 1.启动node程序  
```pm2 start start.js```    
> 2.启动并指定程序名  
```pm2 start start.js --name APP```  
> 3.集群模式启动  
-i 标识实例数量 max 表示pm2自动监测可用cpu数量，可手动指定数量  
```pm2 start start.js -i max ```  
> 4.查看某个进程具体情况  
```pm2 describe app```  
> 5.查看进程资源消耗情况  
``` pm2 monit ```

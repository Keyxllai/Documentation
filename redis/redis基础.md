Redis基础
============
## 概述
Redis【remote dictionary server】是开源高性能的key-value存储，用于构建高并发web程序，如秒杀扣减库存，用户高访问量，热搜等场景。  
- 数据完全基于内存，仅持久化才使用磁盘
- 丰富的数据类型【**字符串String，字典Hash，列表List，集合Set，有序集合SortedSet**】
- 支持集群不部署，数据可复制到从服务器
## Redis和Memcached区别
- redis支持更加丰富数据结构，Memcache仅简单的字符串
- redis支持持久化，事务
- 支持数据备份
- value大小，redis达到512MB,Memcache为1MB
- Redis为单线程的IO模型，受CPU性能影响，Memcache多线程。
## 大量Key同时设置到期时间，注意！！！
```
大量Keys同时过期可能在某个时间点redis服务器出现卡顿，大量请求会查询DB，严重可能造成雪崩，注意在设置过期时间加上随机数
```
## 如何获取大量以固定前缀开始的Keys
> 1. keys kk*  

Keys过多会消耗redis服务器内存，可以通过**keys指令**一次性返回所有匹配的keys;但keys过多可能出现卡顿，产线环境慎用。  

> 2. scan cursor  

语法：**SCAN cursor [MATCH pattern][COUNT count]**，例子：**scan 0 match kk* count 100**  


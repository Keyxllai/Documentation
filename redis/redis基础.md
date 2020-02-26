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
## Redis读写快原因
- 完全基于内存，数据在内存完成请求的计算
- 单线程，避免不必要上下文切换和竞争，也不存在多进程或者多线程的切换导致消耗CPU, [**CPU多核，可以单机多个Redis实例**]
- 多路IO复用模型，非阻塞IO

## 大量Key同时设置到期时间，注意！！！
```
大量Keys同时过期可能在某个时间点redis服务器出现卡顿，大量请求会查询DB，严重可能造成雪崩，注意在设置过期时间加上随机数
```
## 如何获取大量以固定前缀开始的Keys
> 1. keys kk*  

Keys过多会消耗redis服务器内存，可以通过**keys指令**一次性返回所有匹配的keys;但keys过多可能出现卡顿，产线环境慎用。  

> 2. scan cursor  

语法：**SCAN cursor [MATCH pattern][COUNT count]**，例子：**scan 0 match kk* count 100**  

## Redis过期策略
> **定期删除+惰性删除**  

**定期删除**：Redis默认每隔100s随机抽取设置了过期时间的Keys去检查，如果过期就删除。  
**惰性删除**：Redis不主动删除过期的Key,当读取某个Key的时候，Redis去检查该Key是否设置过期时间和是否过期，如果过期就删除Key并且不返回任何值，就是所谓的惰性删除。  
结合上面两种策略redis能够保证过期的Key一定能够删除

## Redis内存淘汰机制
背景：假设大量的key没有被redis定期删除策略移除，并且这些key没有被读取，这就会导致大量不会被命中的keys存在内存中，导致redis内存耗尽。这种情况就需要走**内存淘汰机制**。  
**过程：**  
1. 客户端向redis服务器申请内存指令，如Set
2. Redis会检查当前的内存使用情况，如果已使用内存大于maxmemory设置的值，redis就会根据设置的内存淘汰策略来释放内存，删除keys
3. 执行指令
> Redis3.0包含6中淘汰策略  

1. volatile-lru -> remove the key with an expire set using an LRU algorithm
2. allkeys-lru -> remove any key according to the LRU algorithm
3. volatile-random -> remove a random key with an expire set
4. allkeys-random -> remove a random key, any key
5. volatile-ttl -> remove the key with the nearest expire time (minor TTL)
6. noeviction -> don't expire at all, just return an error on write operations  
默认设置为noeviction
> LRU(Least Rencently Used)淘汰

LRU算法根据keys的访问历史记录来淘汰.Redis中每个对象都设置了响应的lru,每次访问都会更新对应的lru.  
Redis中LRU算法是一个近似算法，抽样数默认为5```maxmemory-samples 5```,抽样数越大消耗内存
```
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs a bit more CPU. 3 is very fast but not very accurate.
```
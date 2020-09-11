# 高并发和海量数据下业务实践

## KV缓存
用于缓存用户信息、会话信息、商品信息。参考下面伪代码(python)
```
def get_user(user_id) 
   user = redis.get(user_id)
   if not user
       user = db.get(user_id)
       redis.setex(user_id, ttl, user) // 设置过期时间
    return user
```  
```
def save_user(user)
    redis.setex(user.id, ttl, user)
    db.save_async(user) //异步入库 
```


## 分布式锁
高并发下共享资源使用，抢购活动库存的扣减。参考下面伪代码(typescript)
``` 
deductStock(product:any)
{
    let requestId = new UUID();
    try {
        let result = redis.setnx(product.id, requestId);
        if(!result){
            return '系统繁忙，请重试';
        }
        redis.expire(product.id, 10); //设置lock失效时间,避免宕机早餐死锁饥饿
        let stock = redis.get(product.id)
        if(stock > 0){
            let realStock = stock - 1;
            redis.set(product.id, realStock);
        }
        else{
            return '库存不足';
        }
    } finally (error) {   // 目的避免上锁后业务异常造成锁无法释放的情况
       if(requestId === redis.get(product.id)){ //释放属于自己的锁，避免误解锁
           // 是否锁
           redis.delete(product.id);
       }
    }
}
```
上面属于简单的分布式锁，最大缺点是加锁只作用一个redis节点，但在集群模式主从切换时候存在锁丢失问题，官方推荐RedLock，但是运维成本高，需要3台以上独立redis实例；二是主从切换概率低，即时发生主从切换需要一个过程，这个过程锁通常会过期，也不会发生锁丢失；

## 延时队列
**场景**
> 1 活动结束前1小时前给用户推动消息  
> 2 优惠券过期前1小时给用户推送消息  
> 3 下单半小时后未支付自动取消订单  
> 4 订单评论7天未评论，系统自动产生一条评论  

**解决方案**
1. 扫表  
每隔一段时间扫描数据库整张表，判断是否满足触犯条件。最大问题是有延迟，不能再指定时间里出发，时效性高场景不满足需求。
2. 延时消息队列  
目前有MQ消息队列支持延时消息，如kafka、RabbitMQ. 再推送消息时候可以指定多长时间后再发送到消费者。这个方案开发成本小，需要中间件支持延时消息。最大问题是延时任务需要重新更新时间就做不到，消息发出去了就收不回。
3. 时间片轮询  
开发耗时，自己实现持久化和高可用
4. Redis ZSET实现
zset 里面存储的是 value／score 键值对,可以将value按照score进行排序，而SET是无序的。  
a. 将 value 存储为序列化的任务消息，score 存储为下一次任务消息运行的时间（deadline）```zadd orderid tts value```  
b. 然后轮询 zset 中 score 值大于 now 的任务消息进行处理  
```
while(true) {
    try{
        $msg = $redis->zRangeByScore($delayQueue,0,time(),0,1);
        if($msg){
            continue;
        }
        //删除消息
        $ok = $redis->zrem($delayQueue,$msg);
        if($ok){
            //业务处理
        }
    } catch(\Exception $e) {

    }
}
```
这里又产生了一个问题，同一个任务可能会被多个进程取到之后再使用 zrem 进行争抢，那些没抢到的进程都是白取了一次任务，这是浪费。解决办法：将 zrangebyscore和zrem使用lua脚本进行原子化操作  
```
local res = nil
local tasks = redis.pcall("zrevrangebyscore", KEYS[1], ARGV[1], 0, "LIMIT", 0, 1)
if #tasks > 0 then
  local ok = redis.pcall("zrem", KEYS[1], tasks[1])
  if ok > 0 then
    res = tasks[1] 
  end
end
return res
```
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

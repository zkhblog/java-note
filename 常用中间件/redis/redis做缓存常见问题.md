***
生产上你们的内存设置多少？
如何配置，修改redis的内存大小？
如果内存满了你怎么办？
redis清理内存的方式？定期删除和惰性删除了解过吗？
手写LRU算法？
***

# 缓存和数据库一致性问题说明
1 场景说明：  
通常在开发中，都会使用mysql作为存储，而redis作为缓存，加速和保护mysql。但是，当mysql数据更新之后，redis怎么保持同步呢。
强一致性同步成本太高，如果追求强一致，那么没必要用缓存了，直接用mysql即可。通常考虑的，都是最终一致性  
2 解决方案：  
常用更新数据库后删除缓存，查询时再添加缓存。并且在单体架构中通过采用事务方式保证缓存与数据库的操作同时成功或失败  
```java
@Override
@Transactional
public Result updateShop(Shop shop) {
    Long id = shop.getId();
    if (null==id){
        return Result.fail("店铺id不能为空");
    }
    // 更新数据库
    boolean b = updateById(shop);
    // 删除缓存
    stringRedisTemplate.delete(SHOPCACHEPREFIX+shop.getId());
    return Result.ok();
}
```

# 缓存穿透
指查询一个一定不存在的数据，导致每次请求都要到数据库去查询。如发起查询id为"-1"的数据或id为特别大不存在的数据，这时的用户很可能是攻击者，攻击会导致数据库压力过大。  
解决方式：  
1 缓存空对象：对于每次没有查到的请求，设置一个空缓存  

2 布隆过滤器：将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力

# 缓存雪崩
缓存雪崩是指在我们设置缓存时key采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到数据库，数据库瞬时压力过重雪崩。  
解决方案：  
1 规避雪崩：缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。  
2 给缓存业务添加降级限流策略  
3 给业务添加多级缓存  

# 缓存击穿
缓存击穿问题也叫热点Key问题，就是一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击  
缓存击穿指并发查同一条数据。缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力  
缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库  

解决方案：  
1 设置热点数据永远不过期  

2 互斥锁  
```java
/**
 * 尝试获取锁
 *
 * @param key
 * @return
 */
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    return BooleanUtil.isTrue(flag);
}

/**
 * 删除锁
 *
 * @param key
 */
private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```
```java
public Shop queryWithMutex(Long id) throws InterruptedException {
    //从Redis查询商铺缓存
    String cacheShop = stringRedisTemplate.opsForValue().get(SHOPCACHEPREFIX + id);

    //判断缓存中数据是否存在
    if (!StringUtil.isNullOrEmpty(cacheShop)) {
        //缓存中存在则直接返回
        try {
            // 将子字符串转换为对象
            Shop shop = objectMapper.readValue(cacheShop, Shop.class);
            return shop;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }

    // 因为上面判断了cacheShop是否为空，如果进到这个方法里面则一定是空，直接过滤，不打到数据库
    if (null != cacheShop) {
        return null;
    }

    Shop shop = new Shop();
    // 缓存击穿，获取锁
    String lockKey = "lock:shop:" + id;
    try {
        boolean b = tryLock(lockKey);
        if (!b) {
            // 获取锁失败了
            Thread.sleep(50);
            return queryWithMutex(id);
        }
        //缓存中不存在，则从数据库里进行数据查询
        shop = getById(id);

        //数据库里不存在，返回404
        if (null == shop) {
            // 缓存空对象
            stringRedisTemplate.opsForValue().set(SHOPCACHEPREFIX + id, "", 2, TimeUnit.MINUTES);
            return null;
        }
        //数据库里存在，则将信息写入Redis
        try {
            String shopJSon = objectMapper.writeValueAsString(shop);
            stringRedisTemplate.opsForValue().set(SHOPCACHEPREFIX + id, shopJSon, 30, TimeUnit.MINUTES);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    } catch (Exception e) {

    } finally {
        // 释放互斥锁
        unLock(lockKey);
    }
    //返回
    return shop;
}
```

# redis宕机
1 事发前  
通过主从架构和哨兵机制尽量保证整个 redis 集群的高可用性，发现机器宕机尽快补上，选择合适的内存淘汰策略...  

2 事发中：本地ehcache缓存 + hystrix限流&降级  
① 对于一个请求，先查本地ehcache缓存，如果没查到再查redis。 如果redis和ehcache都没有，再查数据库，将数据库中的结果，写入ehcache和redis中  
② 限流组件，可以设置每秒的请求，有多少能通过组件，剩余的未通过的请求可以走降级，可以返回一些默认的指，或者友情提示  
***这样做的好处是保证数据库不会挂掉，只要数据库不死，对用户来说，多点几次，至少能刷出一次页面来***

3 事发后：利用 redis 持久化机制保存的数据，重启服务，尽快恢复缓存  


①单机版的加锁
加syncroized，如果线程不释放锁，会造成线程积压的后果，俗称不见不散
加reentrantlock，可以调用trylock(指定时间)，如果在指定时间内加锁不成功，放弃操作，俗称过时不候

②用redis的setIfAbsent()加分布式锁，如果没有线程加锁，可以加锁成功，用完之后删除这把锁stringRedisTemplate.delete(REDIS_LOCK);
加锁代码：
string value=UUID.randomUUID().toString()+Thread.currentThread().getName();
Boolean flag=stringRedisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value);
if(!flag){
   return "抢锁失败";
}

③如果出现异常的话，即在delete之前出现异常，那么不能正常删除，可能无法释放锁，
必须要在代码层面finally中释放锁，保证程序无论是正常执行还是异常执行都可以正常的删除并释放锁

④部署了服务的机器挂掉了，代码层面根本没有办法走到finally块中，
没有办法保证解锁，所以这个key没有被删除，所以为了保证这个即使服务挂了也一定会被删除，
需要在这个key上加一个过期时间限定

⑤设置锁和给key设置过期时间分开了，必须合并成一行才具有原子性
在这里可以使用特定api可以保证原子性，
stringRedisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,value,10L,TimeUnit.SECONDS);

⑥若线程A工作时间超过10秒钟，线程A在redis上加的锁会被redis服务器删除掉，线程B又会加上一把锁，
等线程A工作完之后，会去删除线程B加的锁，所以在这里需要去判断然后再去删除自己线程加的锁
if(stringRedisTemplate.opsForValue.get(REDIS_LOCK).equalsIgnoreCase(value)){
   stringRedisTemplate.delete(REDIS_LOCK);
}

⑦finally块中的判断和删除不是原子性的，否则当加锁与解锁不是同一个客户端时，若在此时，这把锁突然不是这个客户端的，
则会误解锁。为了保证原子性，会使用lua脚本，把finally块中的判断改成lua脚本，保证原子性删除。
除了用lua脚本，还可以使用redis自身的事务特性来保证判断和删除的是原子性操作
while(true){
	stringRedisTemplate.watch(REDIS_LOCK);
	if(stringRedisTemplate.opsForValue().get(REDIS_LOCK).equalsIgnoreCase(value))
	{
		stringRedisTempalte.setEnableTransactionSupport(true);
		stringRedisTemplate.multi();
		stringRedisTemplate.delete(REDIS_LOCK);
		List<Object> list = stringRedisTemplate.exec();
		if(list==null){continue;}
		stringRedisTemplate.unwatch();
		break;
	}
}

Redis调用lua脚本通过eval命令保证代码执行的原子性
将finally块中的代码改成如下：
Jedis jedis = RedisUtils.getJedis();
String script="lua脚本";
try
{
Object o=jedis.eval(script,Collections.singletonList(REDIS_LOCK),Collections.singletonList(value));
if("1".equals(o.toString())){
	syso("ok");
}else{
	syso("error");
}
}finally{
if(null!=jedis){
	jedis.close();
}
}

⑧还存在两个问题
第一个是如何确保redisLock的过期时间大于业务执行时间的问题，也就是redis的分布式锁如何续期，
第二个问题是在集群环境下异步复制会造成锁丢失，也就是master还没来得及将set进去的数据同步到slave中，
master就挂了，此时slave中没有之前的锁。
***
redis的特性是AP，A是高可用，P是分区容错
zookeeper的特性是CP,C是一致性，
因此在锁丢失的问题中，zookeeper的表现要比redis好，但是在高并发下，为了系统的性能，一般选择高可用特性
之后出现的错误问题可以采用人工修理数据的方式来弥补
***

⑨用redisson可以解决
RLock redisson = redisson.getLock(REDIS_LOCK);
redisson.lock();
{
业务逻辑代码
}
finally{
redisson.unlock();
}

代码完善：
在超高并发下，可能会出现解锁的和加锁的不是同一个线程，会报not locked by current thread by node id:***
if(redissonLock.isLocked() && redissonLock.isHeldByCurrentThread()){
   redissonLock.unlock();
}
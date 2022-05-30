1、现在有一个线程正在表t上请求或者持有MDL写锁，把select语句堵住了。
处理方式：就是找到谁持有MDL写锁，然后把它kill掉
2、等flush操作
对表执行flush操作的用法：
flush tables t with read lock;（只关闭表t）
flush tables with read lock;（关闭MySQL里所有打开的表）
3、等行锁
事务占有写锁还不提交，在mysql5.7里，可以通过sys.innodb_lock_waits查看，之后kill处理
4、扫描行数多，执行慢
5、select * from t where id=1这个语句是一致性读，需要从1000001开始，依次执行undo log，执行了100万次以后，才将1这个结果返回；
lock in share mode的SQL语句，是当前读，因此会直接读到1000001这个结果
for update也是当前读，并且加上写锁
6、 select * from table_a where b='1234567890abcd';
在传给引擎执行时，会做字符串截断处理，因为引擎里面这个字段定义的长度是10，将满足条件的所有数据
查出整行做回表处理，全部拿到server层再次判断b值


对“幻读”的说明：
①在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的。因此，幻读在“当前读”下才会出现。
②上面session B的修改结果，被session A之后的select语句用“当前读”看到，不能称为幻读。幻读仅专指“新插入的行”。
幻读的问题：
①语义上冲突
②


binlog保证数据不丢失
write 和fsync的时机，是由参数sync_binlog控制的：
sync_binlog=0的时候，表示每次提交事务都只write，不fsync；
sync_binlog=1的时候，表示每次提交事务都会执行fsync；
sync_binlog=N(N>1)的时候，表示每次提交事务都write，但累积N个事务后才fsync。

redolog保证数据不丢失
redolog的写入策略，通过innodb_flush_log_at_trx_commit参数控制
1、设置为0的时候，表示每次事务提交时都只是把redo log留在redo log buffer中;
2、设置为1的时候，表示每次事务提交时都将redo log直接持久化到磁盘；
3、设置为2的时候，表示每次事务提交时都只是把redo log写到page cache。







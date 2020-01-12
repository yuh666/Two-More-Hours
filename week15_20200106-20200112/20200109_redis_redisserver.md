## redisServer

1. redisDb[] db:保存所有db服务器的数组,比如默认为16个db下标为0-15

2. dbnum:服务器的数量

3. ```
   -- 切换数据库
   select dbIndex
   ```

## redisClient

1. redisDb currentUseDb:记录当前使用的db,指向db的指针 进行select切换时将指针地址进行修改

## redisDb

1. dict *dict:存储数据库中键值对数据,键都为stringObject对象,值通过对应的type的底层去实现

2. dict *expire:存储带有过期时间的键,值存储对应的过期时间的毫秒时(比如expire 100,实际为当前时间+100对应的毫秒值即到什么时间过期)

3. 读写键值对的过程中,键在空间中命中次数

4. ```redis
   info stats
   total_connections_received:2	客户端的连接数
   total_commands_processed:47		总命令数			
   instantaneous_ops_per_sec:0
   total_net_input_bytes:1480
   total_net_output_bytes:26877
   instantaneous_input_kbps:0.00
   instantaneous_output_kbps:0.00
   rejected_connections:0			拒绝的连接
   sync_full:0
   sync_partial_ok:0
   sync_partial_err:0
   expired_keys:0							过期key的数量
   expired_stale_perc:0.00
   expired_time_cap_reached_count:0
   evicted_keys:0
   keyspace_hits:4							获取key时命中的次数
   keyspace_misses:1						获取key时未命中的次数
   pubsub_channels:0
   pubsub_patterns:0
   latest_fork_usec:845
   migrate_cached_sockets:0
   slave_expires_tracked_keys:0
   active_defrag_hits:0
   active_defrag_misses:0
   active_defrag_key_hits:0
   active_defrag_key_misses:0
   ```

## 生存时间或过期时间

1. 设置多久后过期

   ```java
   expire key 100  --设置key 100s之后过期
   pexpire key 100 --设置key 100ms之后过期
   expireat key 23423423423 --设置key到23423423423s值对应的时间过期
   pexpireat key 23423423423000 --设置key到23423423423000ms值对应的时间过期
   ```

2. 最后的底层实现为转换为pexpireat命令,比如

   ```java
   expire key 100
   expire key 100000
   100000+now()=11111100000
   最后设置
   pexpireat key 11111100000时间过期
   ```

3. 通过ttl来查看时间

   ```java
   ttl key
   pttl key
   1.当key不存在返回-2
   2.key存在key没有设置过期时间返回-1
   3.key还未过期返回对应的时间,key过期了返回-2或-1根据key是否存在来判断
   ```

4. 删除过期时间

   ```
   persist message
   ```

## 过期策略

1. 定时删除:通过配置timer定时器,当key到某个时间点过期时自动删除
   - 缺点:当内存够用时,某个时间点过期key特别多此时cpu需要来处理过期key影响客户端的写入性能
   - redis中时间定时器实现为无序链表,查看一个事件为O(N)不能高效处理大量时间
2. 定期删除:在指定时间内去轮询db,记录每次检查删除到的db位置(方便下次从那个开始),每个db中随机获取带有过期时间的key,默认每个数据库每次删除20个后轮到下一个db,如果db无过期key跳过;删除一遍db后记录当前数据db位置的计数器重置当前删除的db为0
3. 惰性删除:当客户端获取当前key的值时进行判断
   - 判断当前key是否存在,不存在直接返回nil
   - 当前key存在,判断expire字典是否存在当前key,不存在则返回数据
   - 存在,判断key是否过期比较当前时间和过期时间,返回对应的过期情况如果过期了则回收(即惰性删除)

## aof和rdb对过期键的处理

1. 生成rdb文件时,对数据库中的键进行检查未失效的key保存
2. 载入rdb文件,如果是主库进行检查;如果是从库全量载入,因为从库加载rdb文件后也会被主从同步数据时先清空从库的数据接收主库的full sync
3. 生成aof文件时,不对key进行检查;当key被删除时,会往aof文件中写入一条del key指令
4. 载入aof文件时,对key进行检查
5. 主从服务器复制时,从服务器只有接收到主服务器的del key才会删除过期的key否则不删除,直接返回键对应值的结果,因为要保证主从数据一致性
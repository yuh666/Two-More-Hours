## redis stats

1. redis-cli info server:服务运行的环境
   - redis_version:redis的版本
   - run_id:运行id
   - redis_mode:使用的模式比如单机版:standalone sentinel:哨兵集群 cluster:cluster集群
   - tcp_port:端口号
2. redis-cli info clients:连接redis server的客户端
   - connected_clients:客户端数量
3. redis-cli info memory:内存使用情况
   - used_memory_human:操作系统分配的内存
   - used_memory_rss_human:消耗的内存,可以通过top指令来查看
   - used_memory_lua_human:lua脚本使用的内存
4. redis-cli info Persistence:持久化信息aof和rdb
   - rdb_last_save_time:上一次触发bgsave的时候,用来判断save表达式是否满足
   - aof_last_rewrite_time_sec:aof上一次进行rewrite操作的时间
5. redis-cli info stats:服务器的状态
   - keyspace_hits和keyspace_misses:缓存命中和miss次数
   - expired_keys:过期key的数量即redis-sever维护的字典expire的元素个数
   - instantaneous_ops_per_sec:当前实例的每秒操作次数即qps最高可以达到10w/s
   - sync_partial_err:半同步复制失败的次数(当失败很多次需要调节复制积压缓冲区的大小)
6. redis-cli info replication:主从赋值相关信息
   - role:当前节点的角色
   - master_repl_offset和slave_repl_offset:master和slave的offset
   - master中存储slave的ip和端口,offset slave中同时存储master的相关信息进行复制使用
   - repl_backlog_size:复制积压缓冲区的大小



## 其他指令排查问题

1. 慢日志的查询:slowlog get
   - slowlog-log-lower-than:查询时间超过多少微秒则加入到slowlog中
   - slowlog-max-len:保存多少条慢日志
2. 最大key占用的大小和各类型的key占用的空间
   - redis-cli --bigkeys,在通过scan去找到key对应的value
3. 官方文档
   - https://redis.io/commands/info
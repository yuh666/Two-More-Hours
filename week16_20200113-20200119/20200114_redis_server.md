## 服务器

## 命令请求执行流程

1. 比如客户端接收命令set key value,首先客户端将命令转换成指定协议的格式,比如*3\r\n.....value\r\n
2. 客户端将协议的数据发送给服务端
3. 服务端监听套接字的io多路复用处理器->文件派发事件处理器->事件处理器(连接 接收 回复)->接收到协议格式中的数据
4. 服务端将协议格式的数据存储到对应的redisclient的query_buf中
5. 服务端读取query_buf中的数据解析后赋值redisclient中的argv和argc参数
6. 根据argv[0]即set命令到对应的命令表(command table)查找命令对应的cmd属性
7. 命令执行时检查redis cmd指针是否指向null,是则返回错误,不是则执行第8步
8. 继续检查cmd中对应的arity属性,判断参数格式是否和命令对应比如 set key value1 value2格式错误
9. 检查客户端是否通过权限验证
10. 服务器是否打开maxmemory,打开后需要先检查内存占用情况,必要时回收后再执行命令
11. 判断客户端能否执行此类命令比如客户端正在使用subscribe命令订阅频道,只能执行subscribe psubscribe等



## 命令执行后续操作

1. 是否开启慢日志操作,开启后需要判断当前命令是否是慢日志是则加入到慢日志日志文件中
2. 命令执行的耗时通过milliseconds属性存储
3. 是否开启aof,开启则添加命令到aof_buf中
4. 从服务器是否正在复制,在复制咋传播给从服务器



## 命令回复客户端

1. 执行完命令回复处理器事件将执行命令的结果写入到客户端的回复buf中比如回复"+ok\r\n"
2. 当客户端端接收回复的内容后,客户端对内容进行解析后返回给用户即ok



## serverCron函数

1. redis维护系统时间到本地缓存,unixtime和mstimes

2. 如果对时间要求较高时调用函数获取机器最新时间比如expire过期时间,时间要求不高的比如key对应的idletime

3. 更新服务器的时钟lruclock(10s更新一次),每个redisobject对象内部会维护lru:22 两者时间差即空闲时间

4. 更新服务器每秒执行命令的次数,通过info stats查看或者info查看所有信息

   ```java
   info stats
   # Stats
   total_connections_received:4
   total_commands_processed:101858
   instantaneous_ops_per_sec:0 --平均每秒中处理命令的次数即qps
   total_net_input_bytes:4712323
   total_net_output_bytes:438364
   instantaneous_input_kbps:0.00
   instantaneous_output_kbps:0.00
   rejected_connections:0
   sync_full:0
   sync_partial_ok:0
   sync_partial_err:0
   expired_keys:0
   expired_stale_perc:0.00
   expired_time_cap_reached_count:0
   evicted_keys:0
   keyspace_hits:50917
   keyspace_misses:1
   pubsub_channels:0
   pubsub_patterns:0
   latest_fork_usec:0
   migrate_cached_sockets:0
   slave_expires_tracked_keys:0
   active_defrag_hits:0
   active_defrag_misses:0
   active_defrag_key_hits:0
   active_defrag_key_misses:0
   ```

5. 更新服务器当前使用的内存数量

   ```java
   info memory
   # Memory
   used_memory:861432
   used_memory_human:841.24K
   used_memory_rss:5943296
   used_memory_rss_human:5.67M
   used_memory_peak:1076144		 -- 当前服务器使用内存1076144b=1076kb=1.03M即_human参数
   used_memory_peak_human:1.03M
   used_memory_peak_perc:80.05%
   used_memory_overhead:847350
   used_memory_startup:797464
   used_memory_dataset:14082
   used_memory_dataset_perc:22.01%
   allocator_allocated:911296
   allocator_active:1171456
   allocator_resident:8617984
   total_system_memory:2095968256
   total_system_memory_human:1.95G
   used_memory_lua:37888
   used_memory_lua_human:37.00K
   used_memory_scripts:0
   used_memory_scripts_human:0B
   number_of_cached_scripts:0
   maxmemory:0
   maxmemory_human:0B
   maxmemory_policy:noeviction
   allocator_frag_ratio:1.29
   allocator_frag_bytes:260160
   allocator_rss_ratio:7.36
   allocator_rss_bytes:7446528
   rss_overhead_ratio:0.69
   rss_overhead_bytes:-2674688
   mem_fragmentation_ratio:7.24
   mem_fragmentation_bytes:5122880
   mem_not_counted_for_evict:0
   mem_replication_backlog:0
   mem_clients_slaves:0
   mem_clients_normal:49694
   mem_aof_buffer:0
   mem_allocator:jemalloc-5.1.0
   active_defrag_running:0
   lazyfree_pending_objects:0
   ```

6. 处理sigterm信号:当服务器关闭时,通过监听该信号来进行数据持久化即安全退出

7. 持久化操作的检查和命令执行流程,服务器维护 rdb_child_pid和aof_child_pid没执行时为-1

   - 有bgrewriteaof被延迟,有执行bgrewriteaof操作
   - 自动报错条件是否满足(比如rdb:save表达式满足执行bgsave
   - 不满足,判断aof重新条件是否满足(即上次aof到本次时间差1s),满足则执行bgrewriteaof操作



## 初始化服务器

1. 服务器的配置初始化:比如端口号 db数量 是否开启rdb aof等
2. 服务器的数据结构初始化,当配置初始化之后server.clients链表,server.db server.lua server.slowlog等属性


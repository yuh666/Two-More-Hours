## 哨兵

1. 哨兵sentinel保证redis集群高可用的一种方案,由一个或者多个sentinel实例组成的sentinel system可以监视一个或多个主服务器(即监视多个集群),同时通过主服务器的info replication得到从服务器信息并监视,当主服务器线下时会选择一台从服务器作为主服务器,并给其他从服务器发送slaveof master_ip master_port让其跟随新的主服务器



## sentinel init

1. 初始化服务器:跟初始化一个普通的redis服务器一样,见14_服务器,但不用去加载数据等
2. 普通redis的代码替换为哨兵使用代码:比如端口号使用哨兵代码定义26379,服务端认可的操作命令表变为哨兵命令表,比如ping 没有了set lpush等指令哨兵不提供写数据功能只是监视
3. 初始化sentinel状态:sentinelInstance中维护master字典表,key:监听主服务器名称比如mymaster值指向sentinelRedisInstance
4. 根据给定的配置文件,初始化sentinel监视主服务器的列表(即初始化sentinelRedisInstance),包括主从节点
   - flags:当前节点状态,SRI_MASTER(主) SRI_SLAVE(从)
   - name:实例的名称格式为ip:port,主服务器可以自己设置
   - runid:实例允许id
   - config_epoch:配置纪元用于失败转移
   - mstime_t down_after_period:实例多久无响应才会判断主观下线
   - quorum:判断该实例为客观下线需要的投票数
   - sentinel address:实例的地址 包括ip和port两个属性
   - slavers:字典存储从服务器列表,key:从服务器name 值:从服务器address(ip+port)
5. 创建连续主服务器的网络连接
   - 命令连接:用于向服务器发送命令,并接受回复;此时哨兵就是redis服务器的客户端
   - 订阅连接:订阅主服务器的_sentinel__:hello频道;因为redis的发布/订阅消息不会保存在redisServer中,当发布一个消息时如果redisclient不在线则接受不到这个消息该消息则丢失了,所以专门用一个连接来监听



## sentinel获取服务器信息

1. 获取主服务器的信息
   - 每10秒向主服务器发送info指令来获取主服务器的信息,存储master相关信息到sentinelRedisSentinel
   - 根据主服务器的replication属性获取从服务器信息存储到主服务器对应的slavers属性中
   - ![image-20200120054047243](/Users/zhouzhihui/Library/Application Support/typora-user-images/image-20200120054047243.png)
2. 获取从服务器的信息
   - sentinel会向从服务器建立命令连接和 订阅连接,每10秒发送info指令来接收从服务器的信息并更新从服务器实例的字典表
   - run_id:允许id   
   - role:角色
   -  master_ip和master_port:主服务器信息 
   - master_link_status:主从服务器的连接状态
   - slave_priority:从服务器的优先级
   - slave_repl_offset:从服务器的复制偏移量
3. 向主服务器从服务器发送命令连接
   - 每两秒一次的频率向主从服务器发送命令连接,publish _sentinel__:hello
   - s_ip s_port s_runid s_epoch:当前哨兵的信息
   - m_name m_ip m_port m_epoch:主服务器的信息
4. 接收主服务或从服务的频道信息
   - 哨兵通过命令连接向sentinel__hello频道发送消息,同时通过订阅sentinel_hello来接收消息,如果该消息不是自己的则处理(通过run_id判断)
   - 否则更新自己的字典表sentinelRedisInstance信息,新增则添加 更新则更新内容
5. sentinel与sentinel之间建立命令连接:用来判断哨兵主观下线和客观下线



## 主观下线和客观下线

1. 主观下线
   - 哨兵以每秒一次的频率向master slave 其他sentinel发送命令连接,在down-after-milliseconds内是否回复
   - 不同哨兵可能判断主观下限的时长不一样,下限后会更新flags标志为SRI_S_DOWN SRI_M_DOWN
2. 客观下线
   - 当哨兵认为一个服务器主观下线后回去询问其他哨兵,当大多人认为主服务器主观下线后主服务器则客观下限进行主服务器的故障失败转移
   - sentinel is_mater_down_by_addr:包括  ip port  current_epoch:*代表客观下线检查 其他代表领导羊的配置纪元
   - 其他哨兵进行回复:down_state:1:下线 0:未下线  leader_runid:*代表检查下线 其他代表领导羊的runid
   - 发送命令的哨兵接收回复消息后进行判断当大多数同意时客观下线



## sentinel 选举

1. 选举遵从raft算法,每个sentinel都可以变为选举节点(局部领头羊),当大多数节点同意变为(领导羊),选举超时重置选举
2. 局部领导羊被认可时当前哨兵节点跟随其后,先到先得即raft cd选举机制
3. 一旦选举产生一个leader(领头羊)之后所有其他节点跟随领头羊,即raft的强领导一致性



## 新主服务选举

1. 删除sentinel字典服务器信息中下线的主服务器和从服务器信息
2. 删除最近5s未恢复info指令的信息
3. 删除所有与主服务器断开超过10s的从服务器,代表从服务器数据不够完整
4. 领头羊对从服务器进行排序,按以下依次比较
   - 优先级高
   - offset完整性
   - run_id最小



## 修改服务器

1. 给新选举出来的从服务器发送slaveof no one指令即断开与宕机的主服务器直接的连接
2. sentinel给其他从服务器发送slaveof master_ip master_port命令让从服务器连接新的主服务器
3. 当宕机的主服务器重新连接后变为新主服务器的从节点
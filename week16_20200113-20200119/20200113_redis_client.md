## 客户端

1. 服务器通过redisClient来保存客户端的状态信息,在redisserver中通过list链表来存储redisClient信息

2. 客户端分为两种类型:

   - 伪客户端:属性df:-1  代表命令请求来源于aof的重做或者lua脚本(redis启动时会自动创建一个lua脚本的客户端),不需要进行网络请求
   - 普通客户端:属性fd的值大于-1,用来记录客户端的套接字描述符

   ```redis
   -- 查看客户端列表 name:客户端名称
   client list
   id=6 addr=127.0.0.1:51494 fd=8 name= age=9 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client
   ```

3. redisClient中flags,通过标志记录客户端的角色(比如执行啥命令,主从同步啥)

4. querybuf:输入缓存区,记录客户端请求的命令,根据输入内容来动态变化当大于1G时会关闭客户端

5. argv和argc:服务器对客户端querybuf的命令进行解析后数据,argv:代表参数 argc:代表个数

   ```
   set key value
   querybuf: * 3 \r \n .. v a l u e ....
   argv:3
   argc:3
   ```

6. redisCommand cmd:存储命令的实现函数 命令标志 命令应该给定参数个数 执行次数和总耗时

7. buf:输出缓存区,当服务器执行完命令后将数据保存到客户端的输出缓冲区;当输出缓存区不够使用时,使用可变大小的缓存区,右reply链表(一个一个字符串对象组成)

8. 身份验证authenticated:0 未通过身份验证,1:通过身份验证

9. 时间:ctime:记录创建客户端的实际 lastinteraction:客户端空转时间即跟上一次服务器互动的时间间隔


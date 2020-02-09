## 位图

1. redis提供了bit位图数据结构,比如我们存储每个用户一年的签到情况,如果通过hash存储的话每个用户要存储365个键值对,当用户上亿时存储空间肯定不够

2. 我们可以通过redis位图的数据结构365天相当于365位,46个字节就够存储了,通过子节点数组来存储

3. ```java
   比如存储hello 
   h:0b1101000
   e:0b1100101
   l:0b1101100
   l:0b1101100
   o:0b1101111
   通过setbit来存储比如存储h即
   setbit s 1 1
   setbit s 2 1
   setbit s 4 1
   get s返回h
   getbit s 3 返回0
   getbit s 4 返回1
   bitcount s 统计1的个数即3
   bitcount s 0 0 统计s中第一个字符1的位数
   ```

   

## hyperLogLog

1. 当需要统计网页用户的访问量时,同一个用户一天访问多次只算一次即唯一标识去重,当数据量较大时set集合并不合适

2. hyperloglog提供了增加计数和获取计数的方法

3. ```java
   pfadd codehole user1
   pfadd codehole user1
   pfadd codehole user2
   pfcount codehole 返回结果2
   ```

4. 两个hyperloglog可以进行merge操作

5. ```java
   pfadd hyperloglog01 user1
   pfadd hyperloglog01 user2
   pfadd hyperloglog01 user3
   
   pfadd hyperloglog02 user1
   pfadd hyperloglog02 user4
   pfadd hyperloglog02 user5
   
   pfmerge hyperloglog01 hyperloglog02 将hyperloglog02中的数据合并到hyperloglog01中
   ```

   


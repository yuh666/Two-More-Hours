## 对象

### redisObject

1. redis中每一个对象都由一个redisObject组成;所有的键总是一个字符串对象,值可以是其他类型的字符串对象比如字符串对象 列表对象 哈希对象 集合对象 有序集合对象,对象的类型由redisObject中type确定

2. redisObject的组成部分

   - type:类型 4个字节
   - encoding:编码类型 4个字节
   - ptr:指向底层数据结构实现的指针

3. type:对象的类型有五种

   - string 字符串对象
   - list 列表对象
   - hash 哈希对象
   - set 集合对象
   - zset 有序集合对

   ```redis
   -- 因为redis中键的类型永远都是字符串类型,所以我们这里说的类型代表键值的redisObject对象类型
   type key 查看键值的redisObject的类型
   ```

4. encoding:编码类型记录对象使用的编码即底层数据结构的实现方案

## 字符串对象

- string类型对象的encoding

  - int long类型的整数

  - embstr embstr编码的简单动态字符串

  - raw 简单动态字符串

  - ```redis
    set stringInt 1   :encoding 对应int类型
    set stringEmbstr abc :encoding 对应embstr类型  当字节小于等于39个字节时
    set stringRaw  大于39个字节的长度:对应raw类型
    ```

  - 注意事项:类型的转换当进行操作时当前数据类型无法满足后向上去转型为embstr或raw

- int类型
  ![image-20200107131448700](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/4382B521981547F4BA793F609BA90C69/9019)

- embstr类型(同时分配redisObject和ptr指向的sdshdr类型)
  ![image-20200107131633896](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/31DFB3B547644A06A433275DCA98EC8E/9021)

- raw类型

  ![image-20200107131704708](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/8B7FCE7D18F54CCBB17FAE237AE6FB60/9020)

## 列表对象

1. 对象编码encoding:ziplist和linkedlist

2. ziplist
   ![image-20200107132010432](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/67AE37A4605A4581BB5825A600C172DC/9023)

3. linkedlist
   ![image-20200107132459146](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/26AA75463F0E4C74A23BF709E2DC1C75/9025)

4. 什么时候从ziplist转为linkedlist

   - 当列表对象中元素超过512个时
   - 列表中存在元素的长度大于64个字节
   - 相关参数:list-max-ziplist-size list-max-ziplist-entries

5. redis 3.2之后quicklist

   ![image-20200107135634224](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/D709DE5351FE4AB0B78763AA6DDA5C76/9027)



## 哈希对象

1. encoding底层存储方式有:ziplist和hashtable(字典)

2. ziplist

   ![image-20200107141247714](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/E2569342ACB74BA5A486B01E72B091FE/9029)

3. hashTable
   ![image-20200107141704363](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/10ADE4AB7A3040E2B2F7E15C85C607F7/9031)

4. ziplist什么时候变成hashtable

   - 当哈希表键或者值的字符串大于64个字节时
   - 哈希表中键值对的对象个数大于512个时
   - 相关参数:hash-max-ziplist-entries:512  hash-max-ziplist-value:64

## 集合

1. intset:整数集合存储  hashtable:字典方式存储
   ![image-20200107144350144](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/D22A44954724410FA56E503D28A4095F/9033)
2. 从intset转换为hashtable,从整数变成stringObject,intset元素个数超过512个 相关配置:set-max-intset-entries 512



## 有序集合

1. ziplist:压缩列表来存储

   ![image-20200107145552967](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/D4782AC31C7C40B0AEDF57ED2BC754E0/9035)

2.hashtable+skiplist

![image-20200107145704480](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/E6AC157FEAC141C49C65886F67C77ABB/9037)

1. 什么时候类型转换?
   - 有序集合的元素大小大于128个时
   - 有序集合中保存的元素有大于64个字节
   - 相关参数:zset-max-ziplist-entries:128 zset-max-ziplist-value:64

## 类型检测

```redis
当执行一个redis命令时服务端会对命令进行检查比如
llen list
1.检查list这个key是否存在redis db对应的字典中
2.在redis db中存在后在检查key对应的type类型是否是list 不是则直接返回
3.是list类型调用其函数来返回结果
```

## 多态实现

```redis
1.基于类型的多态  比如del expire等操作适合所有的type类型的操作
2.基于encoding编码的多态 比如llen 在list列表中ziplist和linkedlist 调用其对应的函数
```

## 内存回收

```redis
redis中的对象通过计数引用来实现,比如redisObject中存储set key1 100;set key2 100
对应的100的计数则为2 当计数为0时则回收,每个redisObject中都有refCount属性
object refcount key:查看refcount的值(在4.0之后0-10000存储的为2147483647 10000:1)
```

## 对象共享

```redis
当redis中出现两个键存储的是相同整数类型的字符串对象时共用一个对象,共用步骤:
1.将数据库键的值指向一个现有的值对象
2.将被共享的值对象的引用计数增加1
注意事项:
	1.只有整数类型的字符串对象能被共享,为啥不能共享字符串和其他类型的对象?比如字符串类型,需要比对字节数组中所有元素时间复杂度O(N) 列表:如果元素为N,每个元素又是一个对象O(N^2)
```

## 对象空转的时长

```redis
# 给定键的空转时长
object idletime key
# 当get之后空转时长变为0或其他方式访问 object idletime不清空
```

## 总结命令

```redis
-- 查看redis对象类型:type
type key
-- 查看redis对象的底层实现类型:encoding
object encoding key
-- 查看redis的计数次数
object refcount key
-- 查看redis的空闲时间(当进行缓存淘汰时使用)
object idletime key
```


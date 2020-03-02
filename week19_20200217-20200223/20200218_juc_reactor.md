## reactor模式

1. java的NIO通过非阻塞api让一个线程来处理多个请求

2. reactor模式的结构图

   <img src="https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/1E330E06684F45A08414F420C18594D4/9159" alt="image-20200218062910649" style="zoom:50%;" />

3. 同步事件多路选择器通过select()方法来监听网络事件

4. 每个监听到的网络事件即一个handle,对应的handle会注册和从reactor中删除

5. 每个I/O handle交给一个event handler来处理



## netty的线程模型

<img src="https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/8A59FA1944BA41B0B4719177B9F5D836/9158" alt="image-20200218062910649" style="zoom:50%;" />

1. 一个eventLoop(事件循环)来处理多个客户端的请求即多对一
2. 每个eventLoop交给一个线程来处理即一对一
3. 本质上每个客户端的请求是交给了一个线程来处理,这样每个网络连接处理即是单线程的没有并发问题
4. eventLoopGroup由两个eventLoop组成,一个是bossGroup:处理连接请求 一个是:workerLoop:用来处理读写请求的,因为socket处理网络请求的机制有关,没当处理socket请求时会创建一个新的socket来处理
5. netty处理请求时,首先请求交给boosGroup,处理完之后交给workGroup,workGroup中对应多个eventLoop,通过负载均衡交给其中一个eventLoop来处理(比如轮询算法)
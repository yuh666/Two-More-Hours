# 了解Tomcat APR

## 1.I/O模型有哪几种？
- 阻塞模型
    - 同步阻塞
    - 同步非阻塞
- 非阻塞模型
    - I/O 多路复用
    - 异步I/O

# 2.什么是同步阻塞？
    一个线程对应一个连接，你若不来我就不走
    用户线程read(阻塞)->内核从网卡拷贝数据到内核缓冲区->唤醒用户线程->用户线程把数据拷贝到Jvm（阻塞）
    对应Tomcat org.apache.coyote.http11.Http11Protocol（已经废弃了）
# 3.什么是同步非阻塞?
    一个线程对应N个连接，挨个连接轮询有没有数据，无数据不阻塞
    用户线程read(无数据非阻塞)->内核从网卡拷贝数据到内核缓冲区->唤醒用户线程->用户线程把数据拷贝到Jvm（阻塞）
    这个没有对应
# 3.什么是I/O多路复用?
    对应到JavaNIO模型的Selector模型，一个线程对应一个Selecor，一个Selector对应N个连接，Selector的select方法非阻塞，会返回内核已经准备好的数据
     用户线程select(非阻塞)->获取到内核从网卡拷贝数据到内核缓冲区的数据（这里与同步非阻塞的区别是 这种直接获取的数据 而后者没有数据，数据还用从Socket里面读出来）->用户线程把数据拷贝到Jvm（阻塞）
    对应Tomcat org.apache.coyote.http11.Http11NIOProtocol
    对应Tomcat org.apache.coyote.http11.Http11APRProtocol
# 4.什么是异步I/O?
    对应到JavaNio.2模型。用户线程不对应任何东西，只负责给Channel注册函数，内核会将数据准备好，回调注册的callback
    用户线程注册callback->内核从网卡拷贝数据到缓冲区->内核从缓冲区读取到应用程序指定的buffer->回调应用程序callback
    这个与I/O多路复用的区别就是【select这个操作交给内核了】
    对应Tomcat org.apache.coyote.http11.Http11NIO2Protocol
    
# 说了这么多 APR到底TMD是个啥？
    可以看到APR是属于I/O多路复用模型，和不同NIO一样，看起来并没有异步I/O牛逼 那么为什么推荐呢
    1.APR使用直接内存实现ZeroCopy 其他都不是
    2.APR是JNI调用的C库 网络处理能力强于Java
    3.如果静态资源较多 那么必须使用APR 
# 我是APR(Apache Portable Runtime Libraries) 我为自己带盐 

 
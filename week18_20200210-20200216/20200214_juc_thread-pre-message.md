## thread-pre-message

1. 为每个执行的任务创建一个子线程去处理请求(相当于生活中的委托别人去办理)

   ```java
   final ServerSocketChannel ssc = 
     ServerSocketChannel.open().bind(
       new InetSocketAddress(8080));
   //处理请求    
   try {
     while (true) {
       // 接收请求
       SocketChannel sc = ssc.accept();
       // 每个请求都创建一个线程
       new Thread(()->{
         try {
           // 读Socket
           ByteBuffer rb = ByteBuffer
             .allocateDirect(1024);
           sc.read(rb);
           //模拟处理请求
           Thread.sleep(2000);
           // 写Socket
           ByteBuffer wb = 
             (ByteBuffer)rb.flip();
           sc.write(wb);
           // 关闭Socket
           sc.close();
         }catch(Exception e){
           throw new UncheckedIOException(e);
         }
       }).start();
     }
   } finally {
     ssc.close();
   }   
   ```

2. 存在问题:

   1. bio:存在两个阻塞 获取连接请求时,当不存在连接请求时当前线程阻塞;执行read方法时当客户端未发送数据时当前线程阻塞;所以给每个执行请求的线程分配一个线程去执行read方法这样多个连接请求过来时不会因为前面请求read不到数据阻塞后面线程执行
   2. java中的线程是重量级的,当高并发下每个请求都要单独的去开一个线程来执行,jvm很容易就oom
   3. 解决方案:
      - nio(同步非阻塞),i/o多路复用,通过select的多路轮询复用来保证一个线程可以处理多个请求
      - java的内库,轻量级线程Fiber也叫做协程;java中线程对应的创建是在内核空间中,协程的创建是在用户空间
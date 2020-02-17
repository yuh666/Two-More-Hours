## 线程本地存储

1. 每个线程本地存储自己的操作变量数据,这样多个线程不会共享数据,也就没有线程安全问题



## 局部变量

1. 每个方法的入栈都会创建对应的栈帧,栈帧中存储方法的入口和出口,局部变量;每个线程操作方法都有自己的栈帧,这样变量即避免了共享;缺点:每次方法的执行都会创建对象



## ThreadLocal

1. 原始的方案是:维护一个map,map中key:线程id value:变量的值 不好的地方:因为map的生命周期比线程的生命周期长很多,导致执行完成的thread对象一直不能被回收
2. java中ThreadLocal不持有对象,持有对象的是Thread,Thread中持有ThreadLocal.ThreadLocalMap对象,ThreadLocalMap中的Entry数组,key:ThreadLocal对象 value:值,即一个线程只会持有一个ThreadLocal对象,多次set后值覆盖;同时Entry持有ThreadLocal的弱引用即当ThreadLocal对象被回收时对应的Entry对象也被回收;Thread线程回收时对应的Thread.ThreadLocalMap也被回收
3. 好处:随着线程的销毁而销毁,每个线程持有自己的ThreadLocalMap对象根据ThreadLocal来获取其Entry中值,避免了数据共享;不会造成内存的泄漏;但是在使用线程池的时候,由于线程池中的线程可以被重复的利用导致不能被回收,此时通过try finally中ThreadLocal.remove手动来清空当前线程中的ThreadLocalMap即threadLocals属性
4. 源码分析:https://juejin.im/post/5ce7e0596fb9a07ee742ba79
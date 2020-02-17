## Guarded Supension

1. 保护性暂停地模式:比如dubbo的异步转同步机制,没接收一个请求后交给线程异步去处理,当前线程加入到等待队列,等执行结果完成后唤醒等待的条件队列

2. 由三部分组成

   - guardedObject:受保护的对象
   - get(Predicate<T> p):方法,判断条件是否满足p.test,不满足一直阻塞 满足返回结果
   - onChange(T obj):改变目标对象的状态,唤醒条件等待队列

3. ```java
    
   class GuardedObject<T>{
     //受保护的对象
     T obj;
     final Lock lock = 
       new ReentrantLock();
     final Condition done =
       lock.newCondition();
     final int timeout=1;
     //获取受保护对象  
     T get(Predicate<T> p) {
       lock.lock();
       try {
         //MESA管程推荐写法
         while(!p.test(obj)){
           done.await(timeout, 
             TimeUnit.SECONDS);
         }
       }catch(InterruptedException e){
         throw new RuntimeException(e);
       }finally{
         lock.unlock();
       }
       //返回非空的受保护对象
       return obj;
     }
     //事件通知方法
     void onChanged(T obj) {
       lock.lock();
       try {
         this.obj = obj;
         done.signalAll();
       } finally {
         lock.unlock();
       }
     }
   }
   ```

4. 扩展,当有多个请求条件保护对象的方法时,此时需要发送和回调的保护对象一直性

5. ```java
    
   class GuardedObject<T>{
     //受保护的对象
     T obj;
     final Lock lock = new ReentrantLock();
     final Condition done = lock.newCondition();
     final int timeout=2;
     //保存所有GuardedObject
     final static Map<Object, GuardedObject> gos=new ConcurrentHashMap<>();
     //静态方法创建GuardedObject
     static <K> GuardedObject create(K key){
       GuardedObject go=new GuardedObject();
       gos.put(key, go);
       return go;
     }
     static <K, T> void fireEvent(K key, T obj){
       GuardedObject go=gos.remove(key);
       if (go != null){
         go.onChanged(obj);
       }
     }
     //获取受保护对象  
     T get(Predicate<T> p) {
       lock.lock();
       try {
         //MESA管程推荐写法
         while(!p.test(obj)){
           done.await(timeout, 
             TimeUnit.SECONDS);
         }
       }catch(InterruptedException e){
         throw new RuntimeException(e);
       }finally{
         lock.unlock();
       }
       //返回非空的受保护对象
       return obj;
     }
     //事件通知方法
     void onChanged(T obj) {
       lock.lock();
       try {
         this.obj = obj;
         done.signalAll();
       } finally {
         lock.unlock();
       }
     }
   }
   
   --发送消息流程A->B
   1.A创建GuardedObject对象,指定发送和消息规定的messageId
   2.A调用get方法等待B结果反馈
   3.B条件fireEvent方法传入messageId和反馈结果(根据messageId获取受保护的对象A,唤醒A的条件等待队列)
   ```



## 思考题:为啥不用sleep来等待

```java
//获取受保护对象  
T get(Predicate<T> p) {
  try {
    while(!p.test(obj)){
      TimeUnit.SECONDS
        .sleep(timeout);
    }
  }catch(InterruptedException e){
    throw new RuntimeException(e);
  }
  //返回非空的受保护对象
  return obj;
}
//事件通知方法
void onChanged(T obj) {
  this.obj = obj;
}
```

1. sleep需要等待到timeout执行完之后才会被唤醒,未被唤醒之前可能结果已返回
2. sleep没有锁的情况下obj变量的改变没有可见性即等待线程不一定能读取到obj改变可以volatile修饰或加锁
3. 扩展sleep和wait的区别,复习
   - sleep不会释放锁和cpu的执行权,wait会释放锁和cpu执行权
   - sleep是thread的方法,wait是object的方法,必须要有等待队列才能调用wait比如synchronized锁住对象对应的monitor监视器,当调用wait时加入到监视器对应的等待队列中与notify对应使用
   - sleep在执行完timeout睡眠后自动唤醒,wait并行有notify与之对应
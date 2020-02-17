## balking模式

1. 多线程版本的if对比guarded当条件不满足时等待;balking模式下条件不满足则直接返回

2. 比如文件编辑器的自动保存,通过定时任务隔一段时间来保存一次;保存时判断是否修改过,修改过则保存;未修改则不保存

   ```java
   class AutoSaveEditor{
     //文件是否被修改过
     boolean changed=false;
     //定时任务线程池
     ScheduledExecutorService ses = 
       Executors.newSingleThreadScheduledExecutor();
     //定时执行自动保存
     void startAutoSave(){
       ses.scheduleWithFixedDelay(()->{
         autoSave();
       }, 5, 5, TimeUnit.SECONDS);  
     }
     //自动存盘操作
     void autoSave(){
       if (!changed) {
         return;
       }
       changed = false;
       //执行存盘操作
       //省略且实现
       this.execSave();
     }
     //编辑操作
     void edit(){
       //省略编辑逻辑
       ......
       changed = true;
     }
   }
   
   修改:volatile changed=false
     void onchange(){
      changed = true
   }
   或者autoSave方法和onchange方法加锁   
   synchronized(this){
     			
   }
   ```

3. 存在问题:

   - 编辑操作的业务逻辑和并发逻辑未分开 对changed=true抽取为方法
   - 当编辑操作被修改时,多线程下change变量可能未被定时任务执行的线程读取到要保证changed变量的可见性;可通过volatile来修饰或者synchronized加锁的方式来保证



## 思考题

```java
class Test{
  volatile boolean inited = false;
  int count = 0;
  void init(){
    if(inited){
      return;
    }
    inited = true;
    //计算count的值
    count = calc();
  }
}  
```

1. 存在问题,当多个线程同时访问时,inited属性未被初始化,同时执行赋值和count计算
2. 修改方式,判断inited和inited=true的操作加锁
3. inited=true的设置通过cas来保证,保证只有一个线程进行cas操作成功继续执行计算count失败直接返回
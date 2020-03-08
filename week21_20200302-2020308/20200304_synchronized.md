## validate synchronized原理

1. 通过硬件层面来分析处理器的结构->寄存器 写缓冲区 高速缓存(写缓冲器+无效队列 MESI协议,高速缓存对应的flag标记)->可见性 有序性->内存屏障
2. synchronized的原理图:每个锁对象对应一个objectmonitor监视器,entrylist:入口同步队列 waitset:等待队列 cas:count变量 monitor entry和monitor exist指令;释放锁时:flush将写缓冲区数据刷新到高速缓存或主存中,refresh操作将其他处理器中高速缓存或主存中数据读取到自己高速缓存中
![image-20200304071455336](https://note.youdao.com/yws/public/resource/62880f0147b6eccac8a4e534f57f9111/xmlnote/E837D82739C14B7A97EBCBBA41E67A87/9210)


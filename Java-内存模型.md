# Java - 内存模型

## 硬件平台的内存模型
* 程序编译时代码重排, 处理器乱序执行指令
* 不同平台提供的缓存一致性保证强度不同
* 一般通过提供内存屏障等特殊指令来保证数据同步

## JVM内存模型
Happens-Before (JVM保证的偏序关系):
* 在单个线程中保证所有动作按代码顺序发生
* 监视器锁的解锁发生在后续的拿锁操作之前
* 对volatile变量的写操作发生在后续读操作之前
* 新线程的开启发生在该线程中所有动作之前
* 线程中的任意动作发生在其他线程检查到该线程已结束之前
* 中断线程的操作发生在该线程检查到自己被中断之前
* 对象的构造发生在对象执行finalizer之前
* Happens-Before具有传递性

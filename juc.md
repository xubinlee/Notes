# Java.Util.Concurrent(JUC)并发工具包

一、  **CountDownLatch**：计数器闭锁是一个能阻塞主线程，让其他线程满足特定条件下主线程再继续执行的线程同步工具。

使用场景：

1. 并发计算：等待所有线程计算完成之后主线程才能汇总得到最终结果；

2. 模拟并发：可以作为并发次数的统计变量；

二、  **Semaphore** ['seməfɔː]**（信号量）：**控制线程的并发数量。首先初始许可数量new Semaphore(n)，获得许可线程调用acquire()方法将许可数量减1，未获得许可线程阻塞等待，直到其他线程调用release()方法将许可数量加1；

三、  **CyclicBarrier**['saɪklɪk] ['bærɪə]：让一组线程互相等待，当每个线程都准备好之后，所有线程才继续执行。

与CountDownLath的区别：

1. CDL是一个线程等待其他线程，CB是多个线程互相等待；

2. CB的计数器能重复使用，调用多次

四、  **ReentrantLock**[riː'entrənt]：可重入锁，是JDK层级上的一个并发控制工具

与synchronized的区别：

1. 用法：synchronized可加在方法上，也可以加在特定代码块上，而lock需要显示指定起始位置和终止位置；

2. 实现：synchronized是依赖于JVM实现的，而ReentrantLock是JDK实现的；

3. 性能：性能相差无几，底层实现也差不多。但是Android开发synchronized性能偏差；

4. 功能：ReentrantLock功能更强大：

&emsp;&emsp;4.1     ReentrantLock可以指定公平锁或非公平锁，而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁；

&emsp;&emsp;4.2     ReentrantLock提供一个Condition（条件）类，用来实现分组唤醒线程，而synchronized只能要么随机唤醒一个线程要么唤醒全部线程；

&emsp;&emsp;4.3     ReentrantLock提供lockInterruptibly()能够中断等待锁的线程；

**备注**：控制线程同步时，优先考虑synchronized，如果有特殊需要再进一步优化。ReentrantLock如果用的不好，不仅不能提高性能，还可能带来灾难。

五、  **CompletableFuture**：提供异步执行任务的能力，并可以方便的获取处理结果


# 线程

## ThreadLocal

&emsp;&emsp;ThreadLocal用于保存某个线程共享变量：对于同一个static ThreadLocal，不同线程只能从中get，set，remove自己的变量，而不会影响其他线程的变量。

&emsp;&emsp;每个线程内部都维护一个ThreadLocalMap，里边包含若干个Entry（K-V键值对），每次存取都会取到当前线程ID，然后得到该线程对象中的Map，然后与Map交互。

**特别注意：**web容器的线程是重复使用的，web容器使用了线程池，当一个请求使用完某个线程，该线程会放回线程池被其他请求使用，如果使用完ThreadLocal对象而没有手动删掉，那么后面的请求就有机会使用到之前使用过的ThreadLocal对象。**使用ThreadLocal最好是每次使用完就调用remove方法将其删掉，避免先get后set的情况导致业务错误（先get后set不会有问题，因为一个线程同一时刻只能被一个请求使用）。**


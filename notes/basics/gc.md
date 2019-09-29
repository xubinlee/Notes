&emsp;&emsp;在五大区中，有三个是不需要进行垃圾回收的：程序计数器、JVM栈、本地方法栈。因为它们的生命周期是和线程同步的，随时线程的销毁，它们占用的内存会自动释放，所以只有方法区和堆需要进行GC。

**什么是自动垃圾回收：**

&emsp;&emsp;自动垃圾回收是一种在堆内存中找出哪些对象在被引用，哪些对象没被引用，并将后者删掉的机制。

**第一步：标记**

&emsp;&emsp;垃圾回收的第一步是标记。垃圾回收器此时会找出哪些内存在使用中，还有哪些不是。

​             ![](https://github.com/xubinlee/Notes/blob/master/assets/marking.png?raw=true)                                    

**第二步：清除**

​            ![](https://github.com/xubinlee/Notes/blob/master/assets/normal-deletion.png?raw=true)

**第三步：压缩**

&emsp;&emsp;为了提升性能，删除了未引用对象后，还可以将剩下的已引用对象放在一起（压缩），这样就能更简单快捷地分配新对象了。

   ![](https://github.com/xubinlee/Notes/blob/master/assets/deletion-with-compacting.png?raw=true)

**为什么需要分代垃圾收集？**

&emsp;&emsp;逐一标记和压缩 Java 虚拟机里的所有对象非常低效：分配的对象越多，垃圾回收需时就越久。不过，根据统计，大部分的对象，其实用没多久就不用了。

**JVM** **分代**

&emsp;&emsp;存活（没被释放）的对象随运行时间越来越少，大部分对象其实都挺短命的。根据这个规律，就可以用来提升 JVM 的效率了。方法是，把堆分成几个部分（就是所谓的分代），分别是新生代、老年代，以及永生代

**Java是如何判断对象是否需要回收的：**

常见的两种判断的算法：

引用计数算法

可达性分析算法（Java使用的这一种）

**引用计数算法**

引用计数算法是在对象中加入一个计数器，当对象被引用，计数器+1，当引用失效，计数器-1

这种算法实现简单，效率高，但是有一个严重的问题会导致内存泄漏，那就是对象之间循环引用，比如说A对象持有B对象的引用，B对象持有A对象的引用，那么A和B的计数器值永远>=1，也就是说这两个对象永远不会被回收

**可达性分析算法**

Java中定义了一些起始点，称为GC Root，当有对象引用它的时候，就把对象挂载在它下面，形成一个树状结构，当一个对象处于一个这样的树里时，就认为此对象是可达的，反之是不可达，如下图 

   ![](https://github.com/xubinlee/Notes/blob/master/assets/gcroots.png?raw=true)

如图 Object1,Object2,Object3是对象可达的，Object4,Object5,Object6是不可达的

那么有哪些可以作为GC Root呢？

静态属性（被static修饰的属性）

常量（被static final修饰的属性）

虚拟机栈（本地变量表）中引用的对象

本地方法栈中引用的对象
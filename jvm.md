## § JVM内存结构

 ![1569562797818](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1569562797818.png)

![*](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image001.gif)   jvm（HotSpot）由哪些区域组成，每个区域的作用：

**方法区：**线程共享的内存区域，用于存储已被虚拟机加载的类信息、常量、静态变量

**运行时常量池：**方法区的一部分，用于存放编译期生成的各种字面量和符号引用

**程序计数器：**它是一块较小的内存空间，不在Ram上，而是直接划分在CPU上，它的作用是记录当前线程所执行    的字节码的行数。字节码解释工作时，就是通过改变程序计数器的值来选取下一条要执行的指令，分支、循环、跳转等基础功能都是依赖此技术去完成的。

**本地方法栈：**本地方法栈与虚拟机栈作用相似，后者为虚拟机执行Java方法服务，而前者为虚拟机用到的Native方法服务。

**虚拟机栈(Virtual Machine Stack)：**每一个线程都有自己的虚拟机栈，这个栈与线程同时创建，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（StackFrame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。

**堆(Heap)：**存放对象实例

**直接内存（Direct Memory）：**nio使用

 

Java.lang.Runtime 取得一些系统内存信息或者进行垃圾收集处理（GC）

![*](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image001.gif)  jvm区域总体分两类，heap区和非heap区。

heap区又分：Eden Space（伊甸园）、Survivor Space(幸存者区)、Tenured Gen（老年代-养老区）。 

非heap区又分：Code Cache(代码缓存区)、Perm Gen（永久代）、Jvm Stack(java虚拟机栈)、Local Method Statck(本地方法栈)。

![*](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image001.gif)  Java内存空间可划分为：

l   伊甸园区（Eden）：新生的对象都保存在此处，但是这些新生对象不一定会一直存活；

​    |-此处也属于内存空间，如果占满了，就会执行GC处理；

l   旧生代区（Tenured）：如果对象要一直使用，就进入旧生代区，这属于二级回收保险；

​    |-如果要执行GC，肯定先清理伊甸园区，随后发现空间还是不足，继续清理旧生代区；

l   永久区（Perm）：永久区中的数据不会清除，即使程序出现了“OutOfMemoryError”也不会清除；

​    在JDK1.8中元空间（Metaspace）取代了永久区，元空间并不在虚拟机中，而是使用本地内存

调整内存大小：-Xms2048M       -Xmx2048M       -Xmn1024M （通常初始内存和最大内存设置一样）

l   “-Xms”：初始分配的内存大小；默认为物理内存的1/64，但是小于1G；

l   “-Xmx”：最大分配的内存；默认大小为物理内存的1/4，但是小于1G；

l   “-Xmn”：设置年轻代（伊甸园区）的堆内存大小；

 

Sample test = new Sample（）

​                   |                    |     

​         对象引用   		对象本身

**栈存放在一级缓存中，存取速度较快；堆存放在二级缓存中，调用对象的速度相对慢一些**

1. 方法区（method）：又叫静态区，被所有线程共享，JDK1.8之前叫永久区，1.8之后叫元空间

   所有被虚拟机加载的class和方法信息、常量、static变量

2. 堆区（heap）：被所有线程共享

   成员变量、对象本身

3. 栈区（stack）：

   基本类型（使用最频繁的类型，放栈中更加高效，不用每次去new一个对象）、局部变量、对象引用

   栈区分为3个部分：基本类型变量区、执行上下文、操作指令区（存放操作指令）

![*](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image001.gif)  导致JVM进行Full GC的情况及解决办法：

**1.**          **System.gc()方法的调用**

​		建议能不使用此方法就别使用，让虚拟机自己去管理它的内存，可通过通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc()。

**2.**          **老年代空间不足**

​		老年代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误【java.lang.OutOfMemoryError: Java heap space】

为避免以上两种状况引起的Full GC，调优时应尽量做到让对象在Minor GC阶段被回收、让对象在新生代多存活一段时间及不要创建过大的对象及数组。

**3.**          **方法区空间不足**

​		方法区中存放的为一些class的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，方法区可能会被占满，在未配置为采用CMS GC的情况下也会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息【java.lang.OutOfMemoryError: PermGen space】 

为避免方法区占满造成Full GC现象，可采用的方法为增大方法区空间或转为使用CMS GC。

**4.**          **CMS GC时出现promotion failed和concurrent mode failure**

​		对于采用CMS进行老年代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

​		promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入老年代，而此时老年代也放不下造成的；concurrent mode failure是在执行CMS GC的同时有对象要放入老年代，而此时老年代空间不足造成的（有时候“空间不足”是CMS GC时当前的浮动垃圾过多导致暂时性的空间不足触发Full GC）；

对应解决办法为：增大survivor space、老年代空间或调低触发并发GC的比率。

**5.**          **统计得到Minor GC晋升到旧生代的平均大小大于老年代的剩余空间**

**6.**          **堆中分配很大的对象**

​		所谓大对象，是指需要大量连续内存空间的java对象，例如很长的数组，此种对象会直接进入老年代，而老年代虽然有很大的剩余空间，但是无法找到足够大的连续空间来分配给当前对象，此种情况就会触发JVM进行Full GC。
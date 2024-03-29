---
title: JVM入门
date: 2022-09-14 18:00:05
tags: [JVM]
categories: Java基础
---
# JVM入门

### 1、什么是垃圾

- 引用计数算法：为对象添加一个引用计数器，每当对象在一个地方被引用，则该计数器加1；每当对象引用失效时，计数器减1。但计数器为0的时候，就表白该对象没有被引用。
- 可达性分析算法：通过一系列被称之为“GC Roots”的根节点开始，沿着引用链进行搜索，凡是在引用链上的对象都不会被回收。

什么是GC roots

- **Java虚拟机栈中被引用的对象，各个线程调用的参数、局部变量、临时变量等。**
- **方法区中类静态属性引用的对象，比如引用类型的静态变量**
- **方法区中常量引用的对象**
- **本地方法栈中所引用的对象**
- **Java虚拟机内部的引用，基本数据类型对应的Class对象。一些常驻的异常对象**
- **被同步锁（synchronized）持有的对象**



### 2、垃圾回收算法

#### 标记--清除算法

  字面含义，对无效的对象进行标记，然后清除。

优点：哪里不对标哪里后面直接清除就完事，简单快捷。

缺点：进行垃圾回收后，堆空间会有大量的碎片，出现了不规整的情况，在给大对象分配内存的时候由于无法找到足够连续的内存空间，就不得补再一次触发垃圾收集，另外，如果Java堆中存在大量的垃圾对象，那么垃圾回收就必然进行大量的标记和清除动作，这个会造成回收效率的降低。



#### 复制算法

  复制算法就是把Java堆分成两块，每次垃圾回收时只使用其中一块，然后把存活的对象全部移动到另一块区域。

优点。解决了清除算法导致的碎片化空间，

缺点，每次只使用堆空间的一半，造成了java堆空间使用率的下降。



#### 标记整理算法

​	标记--整理算法算是一种折中的垃圾收集算法，在对象标记的过程，和前面两个执行的是一样步骤。但是，进行标记之后，存活的对象会移动到堆的一端，然后直接清理存活对象以外的区域就可以了。这样，既避免了内存碎片，也不存在堆空间浪费的说法了。但是，每次进行垃圾回收的时候，都要暂停所有的用户线程，特别是对老年代的对象回收，则需要更长的回收时间，这对用户体验是非常不好的。

#### 分代算法

​	当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将 java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

**比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。**



### 3、垃圾回收详解

 	先按照图中的操作打印出GC详情数据。JDK1.8 分为年轻代，老年代。元空间。



![image-20220923104626763](https://blog.minkimm.cn/images/mink.jpg)

![image-20220923104724909](http://114.55.238.234:9000/images\image-20220923104724909.png)





当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。GC 期间虚拟机又发现 `allocation1` 无法存入 Survivor 空间，所以只好通过 **分配担保机制** 把新生代的对象提前转移到老年代中去，老年代上的空间足够存放 `allocation1`，所以不会出现 Full GC。执行 Minor GC 后，后面分配的对象如果能够存在 Eden 区的话，还是会在 Eden 区分配内存。

#### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

大对象直接进入老年代主要是为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率

####  长期存活的对象将进入老年代

既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器。

大部分情况，对象都会首先在 Eden 区域分配。如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间（s0 或者 s1）中，并将对象年龄设为 1(Eden 区->Survivor 区后对象的初始年龄变为 1)。

Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的 50% 时（默认值是 50%，可以通过 `-XX:TargetSurvivorRatio=percent` 来设置，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值,可以通过参数 `-XX:MaxTenuringThreshold`

针对 HotSpot VM 的实现，它里面的 GC 其实准确分类只有两大种：

部分收集 (Partial GC)：

- 新生代收集（Minor GC / Young GC）：只对新生代进行垃圾收集；
- 老年代收集（Major GC / Old GC）：只对老年代进行垃圾收集。需要注意的是 Major GC 在有的语境中也用于指代整堆收集；
- 混合收集（Mixed GC）：对整个新生代和部分老年代进行垃圾收集。
- 整堆收集 (Full GC)：收集整个 Java 堆和方法区 


####  

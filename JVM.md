## JVM其他

![img](https://gitee.com/zy0098/image_repo/raw/master/20210823222859.png)

[牛客例题](https://www.nowcoder.com/test/question/done?tid=46203190&qid=252869#summary)

Java体系结构包括四个独立但相关的技术：

- Java程序设计语言
- Java.class文件格式
- Java应用编程接口（API）
- Java虚拟机

我们再在看一下它们四者的关系：

  当我们编写并运行一个Java程序时，就同时运用了这四种技术，用**Java程序设计语言**编写源代码，把它编译成**Java.class文件格式**，然后再在**Java虚拟机中运行class文件**。当程序运行的时候，它通过调用class文件实现了**Java API的方法**来满足程序的Java API调用

![img](https://gitee.com/zy0098/image_repo/raw/master/JVM运行流程.webp)

[牛客例题](https://www.nowcoder.com/test/question/done?tid=46040478&qid=252865#summary)

![preview](https://gitee.com/zy0098/image_repo/raw/master/JVM体系.jpeg)



### 什么是GC

[MinorGC 和 FullGC的理解](https://www.cnblogs.com/leeego-123/p/11298267.html)

垃圾回收(Garbage Collection)是Java虚拟机(JVM)垃圾回收器提供的一种用于在空闲时间不定时回收无任何对象引用的对象占据的内存空间的一种机制。其体现在追踪所有正在使用的对象，并且将剩余的对象标记为垃圾，随后标记为垃圾的对象会被清除，回收这些垃圾对象占据的内存，从而实现内存的自动管理；

如果**不进行垃圾回收，内存迟早都会被消耗完**，垃圾回收可以有效的防止内存泄露，有效的使用可以使用的内存在C/C++中，释放无用变量内存空间的事情要由程序员自己来解决。Java有了GC，就不需要程序员去人工释放内存空间。当Java虚拟机发觉内存资源紧张的时候，就会自动地去清理无用变量所占用的内存空间

没有GC就不能保证应用程序的正常运行。而经常造成STW的GC又跟不上实际的需求，所以才会不断的进行GC优化。



### 内存分配与回收

这两篇博文对于JVM整体更为透彻：  [深入理解JVM](https://blog.csdn.net/gsjwxhn/article/details/107048911)  [备战java虚拟机](https://article.itxueyuan.com/jldmB4)

#### 年轻代与年老代

存储在JVM的Java对象分为两类：

- 一类是生命周期比较短的瞬间，这类对象的创建和消亡都比较迅速
- 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致。
  Java堆区进一步细分的话，分为YoungGen和OldGen。

配置YoungGen和OldGen在堆结构的占比
默认 -XX:NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3。

**在HotSpot中，Eden空间和另外两个Survivor空间省所占比例是8:1:1**。开发人员可以通过-XX:SurvivorRatio=8来调整这个空间比例。
-XX：-UseAdaptiveSizePlicy : 关闭自适应的内存分配策略。

几乎所有的Java对象都是在Eden区被new出来。

绝大部分的Java对象的销毁都是在新生代进行的。

可以使用"-Xmn"设置新生代最大内存大小，这个值一般默认就好了。

#### Minor GC、Major GC、Full GC

GC检索哪些是垃圾时，会导致用户线程暂停，所以希望GC出现的情况少，这里主要对Major GC、Full GC进行调优。因为它们两个GC的时间是Minor GC的10倍以上。

JVM在进行GC时，并非每次都对上面三个内存（新生代、老年代；方法区）区域一起回收，大部分时候回收的是新生代。

针对HotSpot VM的实现，他里面的GC按照回收区域分为两大类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）。

- **部分收集**（Partial GC）

  - 新生代（Eden、S0、S1）进行回收采用（Minor GC/ YGC）
  - 老年代进行回收采用（Major GC/ OGC）
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。

- **整堆收集**（Full GC）

  - 收集整个java堆和方法区的垃圾收集（Full GC）。

##### Minor GC的触发机制：

- 当年轻代空间不足时，就会触发Minor GC，这里的年轻代指得是Eden代满，Survivor满不会触发GC。
- Minor GC非常频繁，一般回收速度也比较快。
- Minor GC会引发STW，暂停其他用户的线程，等垃圾回收线程结束，线程才恢复运行。

##### 老年代GC（Major GC/Full GC）触发机制：

- 指发生在老年代的GC，对象从老年代消失。
- 出现了Major GC，经常会伴随至少一次的Minor GC。
- 如果Major GC后，内存还不足，就报OOM。

##### Full GC触发机制：

- 调用System.gc()时，系统建议执行Full GC，但是不必然执行。
- 老年代空间不足
- 方法区空间不足
- 通过Minor GC后进入老年代的平均大小大于老年代的可用内存。

Java 堆主要分为2个区域-年轻代与老年代，其中年轻代又分 Eden 区和 Survivor 区，其中 Survivor 区又分 From 和 To 2个区

![img](https://gitee.com/zy0098/image_repo/raw/master/20210726162925.png)

### 类加载机制

类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

![img](https://gitee.com/zy0098/image_repo/raw/master/20210726163119.png)

`加载（Loading） 验证（Verification） 准备（Preparation）  解析（Resolution）`

`初始化（Initialization）  使用（Using）卸载（Unloading）`   如果要深入JVM推荐此博文：[备战Java虚拟机](https://article.itxueyuan.com/jldmB4)  [类加载机制-深入理解jvm - 简书](https://www.jianshu.com/p/3556a6cca7e5)

### [双亲委派](https://www.cnblogs.com/taojietaoge/p/10269844.html)

![在这里插入图片描述](https://gitee.com/zy0098/image_repo/raw/master/20210925101030.png)

[牛客例题](https://www.nowcoder.com/test/question/done?tid=46116453&qid=112862#summary)

JDK中提供了三个ClassLoader，根据层级从高到低为：

1. Bootstrap ClassLoader，主要加载JVM自身工作
2. 需要的类。
3. Extension ClassLoader，主要加载%JAVA_HOME%\lib\ext目录下的库类。
4. Application ClassLoader，主要加载Classpath指定的库类，一般情况下这是**程序中的默认类加载器**，也是**ClassLoader.getSystemClassLoader()** 的返回值。（这里的Classpath默认指的是环境变量中配置的Classpath，但是可以在执行Java命令的时候使用-cp 参数来修改当前程序使用的Classpath）

JVM加载类的实现方式，我们称为 **双亲委托模型**：

如果一个类加载器收到了类加载的请求，他首先不会自己去尝试加载这个类，而是把这个请求委托给自己的父加载器，每一层的类加载器都是如此，因此所有的类加载请求最终都应该传送到顶层的**Bootstrap ClassLoader**中，只有当父加载器反馈自己无法完成加载请求时，子加载器才会尝试自己加载。

- 首先从底向上的检查类是否已经加载过，就是这行代码：`Class<?> c = findLoadedClass(name);`
- 如果都没有加载过的话，那么就自顶向下的尝试加载该类。

**双亲委托模型的重要用途是为了解决类载入过程中的安全性问题。    另一方面避免类重复字节码加载，节约内存**

假设有一个开发者自己编写了一个名为***[Java](http://lib.csdn.net/base/java).lang.Object\***的类，想借此欺骗JVM。现在他要使用**自定义ClassLoader**来加载自己编写的**java.lang.Object**类。然而幸运的是，**双亲委托模型**不会让他成功。因为JVM会优先在**Bootstrap ClassLoader**的路径下找到**java.lang.Object**类，并载入它

```java
protected synchronized Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{
   //检查类是否被加载过
    Class c = findLoadedClass(name);
    if(c == null){
        try{
            if(parent != null){
                c = parent.loadClass(name, false);
            }else{
                //父加载器为空默认使用启动类加载器
                c = findBootstrapClassOrNull(name);
            }
        }catch (ClassNotFoundException e){
         //   父类无法处理抛出ClassNotFoundException
        }
        if(c == null){
         //   父类无法处理便调用本身findClass方法进行类加载
            c = findClass(name);
        }
    }
    if(resolve){
        resolveClass(c);
    }

    return c;
}
```

### [Java内存模型](https://www.cnblogs.com/jingmoxukong/p/12101294.html)

![img](https://gitee.com/zy0098/image_repo/raw/master/20210925104534.jpeg)

JMM 定义了 8 个操作来完成主内存和工作内存之间的交互操作。JVM 实现时必须保证下面介绍的每种操作都是 **原子的**（对于 double 和 long 型的变量来说，load、store、read、和 write 操作在某些平台上允许有例外 ）。

- `lock` (锁定) - 作用于**主内存**的变量，它把一个变量标识为一条线程独占的状态。
- `unlock` (解锁) - 作用于**主内存**的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- `read` (读取) - 作用于**主内存**的变量，它把一个变量的值从主内存**传输**到线程的工作内存中，以便随后的 `load` 动作使用。
- `write` (写入) - 作用于**主内存**的变量，它把 store 操作从工作内存中得到的变量的值放入主内存的变量中。
- `load` (载入) - 作用于**工作内存**的变量，它把 read 操作从主内存中得到的变量值放入工作内存的变量副本中。
- `use` (使用) - 作用于**工作内存**的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值得字节码指令时就会执行这个操作。
- `assign` (赋值) - 作用于**工作内存**的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- `store` (存储) - 作用于**工作内存**的变量，它把工作内存中一个变量的值传送到主内存中，以便随后 `write` 操作使用。

volatile 禁止指令重排序（内存屏障） 以及保证可见性  具体可见第二章多线程常用方法volatile部分

**原子性（Atomicity）**

使用Java内存模型直接保证原子性变量包括read、load、assign、use、store和write六个，即大致可以认为基本数据类型的访问、读写都是具备原子性的；

虚拟机为了保证原子性，提供了两个高级的字节码指令 `monitorenter` 和 `monitorexit`隐式使用lock与unlock操作。这两个字节码，在 Java 中对应的关键字就是 `synchronized`。因此，在 Java 中可以使用 `synchronized` 来保证方法和代码块内的操作是原子性的。

**可见性（Visibility）**

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

JMM 是通过 **"变量修改后将新值同步回主内存**， **变量读取前从主内存刷新变量值"** 这种依赖主内存作为传递媒介的方式来实现的。

Java 实现多线程可见性的方式有：

- `volatile` 保证新值能立即同步到主内存，以及每次使用立即从主内存刷新
- `synchronized` 对一个变量执行unlock操作前必须将此变量同步到主内存中（执行store、write操作）
- `final`  被final修饰的字段一旦被初始化完成，且构造器没有把this的引用传递出去（this引用逃逸会导致其他线程访问到初始化一半的对象），那么在其他线程中就能看见final字段的值

**有序性（Ordering）**

有序性规则表现在以下两种场景: 线程内和线程间

- 线程内 - 从某个线程的角度看方法的执行，指令会按照一种叫“串行”（`as-if-serial`）的方式执行，此种方式已经应用于顺序编程语言。
- 线程间 - 这个线程“观察”到其他线程并发地执行非同步的代码时，由于指令重排序优化，任何代码都有可能交叉执行。唯一起作用的约束是：对于同步方法，同步块（`synchronized` 关键字修饰）以及 `volatile` 字段的操作仍维持相对有序。

在 Java 中，可以使用 `synchronized` 和 `volatile` 来保证多线程之间操作的有序性。实现方式有所区别：

- `volatile` 关键字会禁止指令重排序。
- `synchronized` 关键字通过互斥（lock）保证同一时刻只允许一条线程操作。



## JVM高频点

### Ⅰ-① Jvm内存模型 112 

![image-20211014093332116](https://gitee.com/zy0098/image_repo/raw/master/20211014093332.png)

红色区域线程私有不会出现线程竞争关系；蓝色区域线程共享，堆中存对象，方法区存类信息，常量，静态变量等；锁的主要应用范围是数据共享区

[JVM基础](https://www.cnblogs.com/cielosun/p/6622983.html)    [牛客例题](https://www.nowcoder.com/test/question/done?tid=47121893&qid=112835#summary)

**程序计数器**（Program Counter Register）

线程私有，各条线程之间的计数器互不影响，用于在线程执行时充当行号指示器；

程序控制流的指示器，分支，循环，跳转，异常处理，线程恢复等基础功能都依赖这个计数器来完成；

**Java虚拟机栈**（Java Virtual Machine Stack）

线程私有，生命周期与线程相同；为虚拟机执行Java方法服务

Java虚拟机在方法执行时同步创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态连接、方法出口，每个方法被调用直至执行完毕过程对应栈帧在虚拟机栈中从入栈到出栈的过程。

局部变量表：存放编译期间可知的各种Java虚拟机基本数据类型，对象引用和returnAddress类型

- StackOverflowError——线程请求的栈深度大于虚拟机所允许的深度发生栈内存溢出；调整参数`-xss`可以调整JVM栈的大小
- OutOfMemoryError——如过Java虚拟机栈容量可以动态扩展时，栈扩展无法申请到足够内存

**本地方法栈**（Native Method Stacks）

为虚拟机使用到的本地（Native）方法服务； 与虚拟机栈作用相似

与虚拟机一样，在栈深度溢出或栈扩展失败时分别抛出StackOverflowError和OutOfmemoryError异常

**Java堆**（Heap）

线程共享，在虚拟机创建时存放对象实例； 又称GC堆（Garbage Collected Heap）

- 从内存分配角度看，Java堆中可以划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）以提升对象分配时的效率。但不论如何划分都不会改变Java堆中存储内容的共性；
- 可以处在不连续的内存空间中，但逻辑上应该连续
- 在Java堆中没有内存完成实例分配，且堆无法再扩展时，Java虚拟机将抛出OutOfMemoryError异常

**方法区**（Method Area）

线程共享，用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据；逻辑上是堆的一部分，有一个别名叫做非堆（Non-Heap），目的是与Java堆区分开来

方法区受到的约束相对较为宽松，不需要连续的内存、可以选择固定大小或扩展，甚至还可以选择不实现垃圾收集——该区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般而言这个区域的回收效果较难令人满意，尤其是类型卸载的条件相当苛刻，但这部分的回收有时也是必要的。

**运行时常量池**（Runtime Constant Pool）

属于方法区的一部分，Class文件中除了有类的版本，字段，方法，接口等，还有一项信息——常量表（Constant Poll Table），用于存放编译时期生成的各种字面量与符号引用；对于运行常量池《Java虚拟机规范》没有做任何细节要求，不同提供商实现的虚拟机可以按照自己的需求来实现这个内存区域。

[java堆、栈、堆栈，常量池的区别-腾讯云](https://cloud.tencent.com/developer/article/1445731)          [堆，栈，方法区，常量池，的概念-博客园](https://www.cnblogs.com/xiaowangbangzhu/p/10366200.html)



### Ⅱ 垃圾清除相关 （高频）

[参考](https://www.jianshu.com/p/5261a62e4d29)   [参考2](https://www.cnblogs.com/cielosun/p/6674431.html)   [为什么垃圾回收](https://codegym.cc/zh/groups/posts/15-object-lifecycle-)

#### ① Jvm垃圾回收算法  60 

标记清除算法     标记复制算法     分代算法     三色标记算法

前两个重点记忆，分代算法大概知晓新生代老年代关系即可     三色标记暂不明确

[牛客例题](https://www.nowcoder.com/test/question/done?tid=47121893&qid=36411#summary)

  这两篇博文主要讲的垃圾回收： [浅谈GJava垃圾回收机制（GC）](https://www.jianshu.com/p/5261a62e4d29)      [深入理解JVM虚拟机笔记](https://blog.csdn.net/weixin_43691723/article/details/106771107)

在JVM规范中并没有明确GC的运作方式，各个厂商可以采用不同的方式去实现垃圾回收器。垃圾回收算法可划分为“引用计数式垃圾收集”（Reference Counting GC）和“追踪式垃圾收集”（Tracing GC）两类

##### 标记-清除算法（Mark-Sweep）

CMS(Concurrent Mark Sweep)收集器使用此算法，个过程分为四个步骤，包括：

1）初始标记（CMS initial mark） 2）并发标记（CMS concurrent mark）

 3）重新标记（CMS remark） 4）并发清除（CMS concurrent sweep）

又称`老年代垃圾收集器`，最基础的垃圾回收算法，分为两个阶段，标注和清除。标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。如图：  

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719085804.jpeg)

- 优点：不需要进行对象的移动，并且仅对不存活的对象进行处理，在存活对象比较多的情况下极为高效。
- 缺点：（1）***标记和清除过程的效率都不高。***（这种方法需要使用一个空闲列表来记录所有的空闲区域以及大小。对空闲列表的管理会增加分配对象时的工作量）（2）**标记清除后会产生大量不连续的内存碎片。**虽然空闲区域的大小是足够的，但却可能没有一个单一区域能够满足这次分配所需的大小，因此本次分配还是会失败（在Java中就是一次OutOfMemoryError）不得不触发另一次垃圾收集动作

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719085955.webp)

##### 标记-复制算法（ Copying）

又称`新生代回收算法`现在商业虚拟机都采用这种收集算法回收新生代；为了解决Mark-Sweep算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清掉，如图：

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719090209.jpeg)

- 优点：（1）标记阶段和复制阶段可以同时进行。（2）每次只对一块内存进行回收，运行高效。（3）只需移动栈顶指针，按顺序分配内存即可，实现简单。（4）内存回收时不用考虑内存碎片的出现（得活动对象所占的内存空间之间没有空闲间隔）。

- 缺点：需要一块能容纳下所有存活对象的额外的内存空间。因此，可一次性分配的最大内存缩小了一半。

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719090520.webp)

##### 标记-整理算法(Mark-Compact)

`老年代回收算法` 是否移动存活对象都存在弊端，对比标记-清理（产生大量空间碎片），从整个程序吞吐量看移动对象会更划算；CMS收集器采用的“和稀泥”式解决方案，即平时采用标记-清理算法，容忍空间碎片产生直到影响到对象分配时再采用标记-整理算法收集一次获得规整的内存空间

结合了以上两个算法，为了避免缺陷而提出。标记阶段和Mark-Sweep算法相同，标记后不是清理对象，而是将存活对象移向内存的一端。然后清除端边界外的对象。如图：

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719090758.jpeg)

- 优点：（1）经过整理之后，新对象的分配只需要通过指针碰撞便能完成（Pointer Bumping），相当简单。（2）使用这种方法空闲区域的位置是始终可知的，也不会再有碎片的问题了。
- 缺点：GC暂停的时间会增长，因为你需要将所有的对象都拷贝到一个新的地方，还得更新它们的引用地址。

![img](https://gitee.com/zy0098/image_repo/raw/master/20210719091142.webp)

##### 分代收集理论（Generational Collection）

- 弱分代假说（Weak Generational Hypothesis）——绝大多数对象都是朝生夕灭    对应新生代（Young Generation）
- 强分代假说（Strong Generational Hypothesis）——熬过越多次垃圾收集过程的对象就越难以消亡 对应老年代（Old Generation）
- 跨代引用假说（Intergenerational Reference Hypothesis）——跨代引用相对同代引用来说仅占少数；存在互相引用关系的两个对象应倾向于同时生存或同时消亡的

目前大部分JVM的GC对于新生代都采取Copying算法，因为新生代中每次垃圾回收都要回收大部分对象，即要复制的操作比较少，但通常并不是按照1：1来划分新生代。一般将新生代划分为一块较大的Eden空间和两个较小的Survivor空间(From Space, To Space)，每次使用Eden空间和其中的一块Survivor空间，当进行回收时，将该两块空间中还存活的对象复制到另一块Survivor空间中。

![img](https://images2015.cnblogs.com/blog/989246/201704/989246-20170406170311707-1412704605.jpg)

而老生代因为每次只回收少量对象，因而采用Mark-Compact算法。

- 新生代使用：复制算法
- 老年代使用：标记 - 清除 或者 标记 - 整理 算法

另外，不要忘记在[Java基础：Java虚拟机(JVM)](http://www.cnblogs.com/cielosun/p/6622983.html)中提到过的处于方法区的永生代(Permanet Generation)。它用来存储class类，常量，方法描述等。对永生代的回收主要包括废弃常量和无用的类。

对象的内存分配主要在新生代的Eden Space和Survivor Space的From Space(Survivor目前存放对象的那一块)，少数情况会直接分配到老生代。当新生代的Eden Space和From Space空间不足时就会发生一次GC，进行GC后，Eden Space和From Space区的存活对象会被挪到To Space，然后将Eden Space和From Space进行清理。如果To Space无法足够存储某个对象，则将这个对象存储到老生代。在进行GC后，使用的便是Eden Space和To Space了，如此反复循环。当对象在Survivor区躲过一次GC后，其年龄就会+1。默认情况下年龄到达

[Java垃圾回收机制](http://www.cnblogs.com/dolphin0520/p/3783345.html)       [java的gc为什么要分代？ - RednaxelaFX的回答 - 知乎](https://www.zhihu.com/question/53613423/answer/135743258)

##### 三色标记（Tri-color Marking）

三色标记（Tri-color Marking）作为工具来辅助推导，把遍历对象图过程中遇到的对象，按照“是否访问过”这个条件标记成以下三种颜色：

- 白色：`表示对象尚未被垃圾收集器访问过。` 显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达。
- 黑色：`表示对象已经被垃圾收集器访问过。` 且这个对象的所有引用都已经扫描过。黑色的对象代 表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对 象不可能直接（不经过灰色对象）指向某个白色对象。
- 灰色：`表示对象已经被垃圾收集器访问过。`  但这个对象上至少存在一个引用还没有被扫描过。

**三色标记存在问题**

1. 浮动垃圾：并发标记的过程中，若一个已经被标记成黑色或者灰色的对象，突然变成了垃圾，由于不会再对黑色标记过的对象重新扫描,所以不会被发现，那么这个对象不是白色的但是不会被清除，重新标记也不能从`GC Root`中去找到，所以成为了浮动垃圾，**「浮动垃圾对系统的影响不大，留给下一次GC进行处理即可」**。
2. 对象漏标问题（需要的对象被回收）：并发标记的过程中，一个业务线程将一个未被扫描过的白色对象断开引用成为垃圾（删除引用），同时黑色对象引用了该对象（增加引用）（这两部可以不分先后顺序）；因为黑色对象的含义为其属性都已经被标记过了，重新标记也不会从黑色对象中去找，导致该对象被程序所需要，却又要被GC回收，此问题会导致系统出现问题，而`CMS`与`G1`，两种回收器在使用三色标记法时，都采取了一些措施来应对这些问题，**「CMS对增加引用环节进行处理（Increment Update），G1则对删除引用环节进行处理(SATB)。」**

**解决办法**

在JVM虚拟机中有两种常见垃圾回收器使用了该算法：CMS(Concurrent Mark Sweep)、G1(Garbage First) ，为了解决三色标记法对对象漏标问题各自有各自的法；

[三色标记法含动图—掘金](https://juejin.cn/post/6996123086296252453) 		  [JVM三色标记法详解，CMS算法](https://zhuanlan.zhihu.com/p/431406707)



#### ② Jvm垃圾回收器 46 

- 1.serial收集器
  单线程，工作时必须暂停其他工作线程。多用于client机器上，使用复制算法
- 2.ParNew收集器
  serial收集器的多线程版本，server模式下虚拟机首选的新生代收集器。复制算法
- 3.Parallel Scavenge收集器
  复制算法，可控制吞吐量的收集器。吞吐量即有效运行时间。
- 4.Serial Old收集器
  serial的老年代版本，使用整理算法。
- 5.Parallel Old收集器
  第三种收集器的老年代版本，多线程，标记整理
- 6.CMS收集器
  目标是最短回收停顿时间。
- 7.G1收集器,基本思想是化整为零，将堆分为多个Region，优先收集回收价值最大的Region。

##### CMS

CMS全称 `Concurrent Mark Sweep`，是一款并发的、**使用标记-清除算法的老年代垃圾收集器**，在收集过程中可以与用户线程并发操作。它可以与Serial收集器和Parallel New收集器搭配使用。CMS**牺牲了系统的吞吐量来追求收集速度**，适合追求垃圾收集速度的服务器上。可以通过JVM启动参数：`-XX:+UseConcMarkSweepGC`来开启CMS。

1. 初始标记(CMS-initial-mark) ,会导致stw;
2. 并发标记(CMS-concurrent-mark)，与用户线程同时运行；
3. 预清理（CMS-concurrent-preclean），与用户线程同时运行；
4. 可被终止的预清理（CMS-concurrent-abortable-preclean） 与用户线程同时运行；
5. 重新标记(CMS-remark) ，会导致swt；
6. 并发清除(CMS-concurrent-sweep)，与用户线程同时运行；
7. 并发重置状态等待下次CMS的触发(CMS-concurrent-reset)，与用户线程同时运行；

![image-20220214210529108](https://gitee.com/zy0098/image_repo/raw/master/CMS%E5%9E%83%E5%9C%BE%E5%A4%84%E7%90%86.png)

**使用场景：**

GC过程短暂停，适合对时延要求较高的服务，用户线程不允许长时间的停顿。

**缺点：**

服务长时间运行，造成严重的内存碎片化。另外，算法实现比较复杂（如果也算缺点的话）

由于CMS的收集线程和用户线程并发，可能在收集过程中出现"concurrent mode failure"，解决方法是让CMS尽早GC。在一定次数的Full GC之后让CMS对内存做一次压缩，减少内存碎片，防止年轻代对象晋升到老年代时因为内存碎片问题导致晋升失败



[CMS垃圾回收器-博客园](https://www.cnblogs.com/myf008/p/14355711.html)   		[图解CMS垃圾回收器-简书](https://www.jianshu.com/p/2a1b2f17d3e4)

CMS、G1



#### ③ 整体介绍垃圾回收 40 

这里补充下垃圾判断方法

 Java语言规范没有明确地说明JVM使用哪种垃圾回收算法，但是任何一种垃圾回收算法一般要做2件基本的事情：

（1）找到所有存活对象；（2）回收被无用对象占用的内存空间，使该空间可被程序再次使用

[参考](https://codegym.cc/groups/posts/16-more-about-the-garbage-collector-)

##### 方法一：**引用计数算法*****（Reference Counting Collector）\***

 堆中每个对象（不是引用）都有一个引用计数器。当一个对象被创建并初始化赋值后，该变量计数设置为1。每当有一个地方引用它时，计数器值就加1（a = b， b被引用，则b引用的对象计数+1）。当引用失效时（一个对象的某个引用超过了生命周期（出作用域后）或者被设置为一个新值时），计数器值就减1。任何引用计数为0的对象可以被当作垃圾收集。当一个对象被垃圾收集时，它引用的任何对象计数减1。

- 优点：引用计数收集器执行简单，判定效率高，交织在程序运行中。对程序不被长时间打断的实时环境比较有利（OC的内存管理使用该算法）。
- 缺点： 难以检测出对象之间的循环引用。同时，引用计数器增加了程序执行的开销。所以Java语言并没有选择这种算法进行垃圾回收。

```Java
public static void main(String[] args){
    Object object1 = new Object();
    Object object2 = new Object();
    object1.object = object2;
    object2.object = object1;
    object1 = null;
    object2 = null;
}
```

显然，在最后，object1和object2的内存块都不能再被访问到了，但他们的引用计数都不为0（即他们互相引用），这就会使他们永远不会被清除。

##### 方法二：**可达性分析*****根搜索算法（Tracing Collector）\***

![img](https://gitee.com/zy0098/image_repo/raw/master/20210718102545.webp)

上图中 object1~object4对GC Root都是可达的，说明不可被回收，object5和object6对GC Root节点不可达，说明其可以被回收。

现在大多数JVM采用对象引用遍历（***根搜索算法***）；根集(Root Set)就是正在执行的Java程序可以访问的引用变量（注意：不是对象）的集合(包括局部变量、参数、类变量)，程序可以使用引用变量访问对象的属性和调用对象的方法。

##### **GC Roots对象**：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象，譬如各个线程被调用的方法堆中使用到的参数、局部变量、临时变量等

- 在方法区中类静态属性引用的对象，譬如Java类的引用类型静态变量

- 在方法区中常量引用的对象，譬如字符串常量池（String Table）里的引用

- 在本地方法栈中JNI（即通常说的Native方法）引用的对象

- Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象，还有系统类加载器

- 所有被同步锁（synchronized关键字）持有的对象

- 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等

##### **算法的基本思路**：

 （1）通过一系列名为“GC Roots”的对象作为起始点，寻找对应的引用节点。

 （2）找到这些引用节点后，从这些节点开始向下继续寻找它们的引用节点。

 （3）重复（2）。

 （4）搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，就证明此对象是不可用的。

##### Java中的对象引用分类

 （1）强引用（Strong Reference）：如“Object obj = new Object（）”，这类引用是Java程序中最普遍的。只要强引用还存在，垃圾收集器就永远不会回收掉被引用的对象。

 （2）软引用（Soft Reference）：它用来描述一些可能还有用，但并非必须的对象。**在系统内存不够用时**，这类引用关联的对象将被垃圾收集器列为回收范围进行**二次回收**。JDK1.2之后提供了SoftReference类来实现软引用。

 （3）弱引用（Weak Reference）：它也是用来描述非须对象的，但它的强度比软引用更弱些，被弱引用关联的对象**只能生存到下一次垃圾收集发生之前**。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK1.2之后，提供了WeakReference类来实现弱引用。

 （4）虚引用（Phantom Reference）：最弱的一种引用关系，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的是希望能在这个对象被收集器回收时收到一个系统通知。JDK1.2之后提供了PhantomReference类来实现虚引用。

[对象引用与可达性算法说明](https://www.jianshu.com/p/8f5fa8288d9b)

##### finalize()方法

![img](https://gitee.com/zy0098/image_repo/raw/master/20210724090316.png)

运行代价较高，且不确定性较大，**不推荐使用** 对象在GC时唯一被自我拯救的机会，一个对象的finalize方法最多被调用一次

 在根搜索算法中，要真正宣告一个对象死亡，至少要经历两次标记过程：

   1.如果对象在进行**根搜索后发现没有与GC Roots相连接的引用链，那它会被第一次标记并且进行一次筛选**。筛选的条件是此对象是否有必要执行 finalize（）方法（可看作析构函数，类似于OC中的dealloc，Swift中的deinit）。当对象没有覆盖finalize（）方法，或finalize（）方法已经被虚拟机调用过，虚拟机将这两种情况都视为没有必要执行。

   2.如果该对象**被判定为有必要执行finalize（）方法，那么这个对象将会被放置在一个名为F-Queue队列中**，并在稍后由一条由虚拟机自动建立的、低优先级的Finalizer线程去执行finalize（）方法。finalize（）方法是对象逃脱死亡命运的最后一次机会（因为一个对象的finalize（）方法最多只会被系统自动调用一次），稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果要在finalize（）方法中成功拯救自己，只要在finalize（）方法中让该对象重新引用链上的任何一个对象建立关联即可。而如果对象这时还没有关联到任何链上的引用，那它就会被回收掉。

   （4）实际上GC判断对象是否可达看的是强引用。

  当标记阶段完成后，GC开始进入下一阶段，删除不可达对象。

##### 判断类是否应该回收

- 该类的实例化都已被回收，即Java堆中不存在该类及其任何派生子类的实例
- 加载该类的类加载器已经被回收（通常较难完成，存在于精心设计的可替换类场景
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法再任何地方通过反射访问该类的方法

需要同时满足以上三点可以回收



###  Jvm调优参数与实例 14 



###  遇到的OOM问题，如何定位、排查与解决 14 

使用工具jvirsualvm

###  CMS与G1垃圾回收器的区别 12 

CMS分为年轻代和老年代，G1是分为一个个Region

###  如何判断垃圾对象？ 8 

看是否被gc root对象引用

###  堆和栈的区别 8 

栈是存放栈帧，里面有变量表和操作数栈，堆是存放对象的实例

###  可达性分析 7 

从gc root对象根据引用链查看是否引用到这个对象

###  GC Root有哪几种？ 7 

引用对象

###  CMS 垃圾回收的流程 7 

初始标记：会发生停顿，这时候会扫描和gc root对象直接关联的对象

并发标记：不会停顿，在初始标记的基础上继续向下标记

重新标记：并发标记时可能会产生新的对象，重新标记一下，这个时候会停顿

并发清理：清理垃圾对象

###  介绍G1垃圾回收器 4 

G1垃圾回收器里分为了一个个的Region，每个Region可能是Eden区，可能是Suriver区，也可能是old区，可能是Humongous大对象。每次回收会回收n个Region。

###  新生代和老年代的区别 4 

新生代是用来存放刚刚生成的对象，如果这个对象幸存的次数大于阈值或者是大对象，会被放在老年代。如果堆内存不足会优先回收新生代，如果此时新生代内存还是不足则会回收老年代。

###  什么时候发生GC 4 

当新生代满时发生young gc，如果young gc之后新生代内存还是不足，则发生full gc

###  简述栈溢出情况 4 

比如无限制的递归会发生栈溢出，因为方法就是一个栈帧存在栈里。

###  简述栈帧存储的内容 3 

局部变量表、操作数栈

###  对象的生命周期（New出来 到GC ) 3 

创建、使用、不可见、不可达、回收

###  简述什么是内存泄漏 3 

内存中存在多余的对象，却不会被回收。

###  JAVA1.8默认垃圾回收器 2 

cms

###  栈帧是线程私有的吗 2 

是的

###  标记清除和标记整理的区别 2 

标记清除：在标记后直接清除

标记整理：标记后不会立即清除，先把所有幸存对象移动到一端，再从边界开始清除

###  为什么年轻代用复制[算法]() 2 

有一个将幸存对象从from区复制到to区，再把from区清空，这样避免了空间的浪费。

###  jps、jstat、jmap、jstack使用 2 

略

###  为什么要把永久代换成元空间? 元空间使用直接内存的好处 2 

永久代是在堆内存中的，不过元空间是在本地内存中了

使用直接内存只要机器内存够，就不会内存溢出。

###  垃圾对象是否立即回收？ 2 

不会，在内存不足时才会回收。

###  CMS垃圾回收器为什么进行两次停顿？ 2 

第一次停顿是在初始标记阶段，此时是为了标记和gc root对象直接关联的对象，第二次停顿是在重新标记阶段，此时是这了标记在并发标记阶段新产生的对象。

###  某些垃圾回收器为什么需要STW(Stop The World)？ 2 

因为在获取根节点时必须要是在一个保证一致性的快照中才能进行。

###  频繁gc会有什么问题 2 

会造成程序非常卡顿，因为gc会发生stw

###  简述虚拟机栈包含哪些内容 2 

栈帧，栈帧里有局部变量表和操作数栈

###  GC一定会发生停顿吗? 1 

看具体是哪个垃圾回收器

###  JVM能够提供多大内存？ 1 

不造。

###  新生代为什么分为Eden，From Survive，To Survive三个区 1 

因为是复制清除算法

###  Full GC 过多如何排查分析? 1 

可以通过工具查看内存信息

###  简述Minor GC 与 Full GC及其区别 1 

新生代的gc和新生代老年代一起的gc

###  Jvm1.7 与 1.8的区别 1 

？

###  线程共享的位置都哪些 1 

方法区、堆

###  垃圾回收为什么要分区？ 1 

因为有些对象是常用对象不用一直检测

###  新生代为什么设计eden与sur[vivo]()r两个区？ 1 

因为有此对象是幸存的，可能会晋升到old

###  CMS并发标记时，那些线程被允许执行？ 1 

用户线程

###  三色标记法 1 

白色：尚未被扫描的对象

灰色：正在被扫描中的对象

黑色：已被扫描的对象，并且被标记

###  哪些垃圾回收器需要STW(Stop The World)? 1 

cms、g1

###  实例变量能否作为GC root 1 

不行，要是变量的引用

### 

###  内存模型中堆和栈的区别 1 

栈：用来存放栈帧

堆：用来存放对象的实例

###  CMS产生空间碎片后，将进行什么操作？ 1 

对于存活对象进行移动整理

###  对象进入老年代的条件 1 

存活对象的存活次数达到阈值

对象的大小超过了新生代一半

###  Jvm哪些内存区域不会产生内存溢出 1 

程序计数器

 "如果触发新生代GC时，存活对象总量大于sur[vivo]()r区容量，JVM如何处理？" 1 

直接进入老年代

###  新生代为什么默认设置年龄为15？ 1 

因为记录这个年龄的空间是4位的，最多只能记录到15

###  如果任务过多，线程池的阻塞队列会撑爆哪部分内存区域？ 1 

如果使用无界队列，任务过多可能会产生oom，会撑爆堆内存

###  JDK9采用哪种垃圾回收[算法]()？ 1 

G1

###  如何理解CMS垃圾回收器对CPU资源敏感？ 1 

因为CMS垃圾回收器是和用户线程并发执行的，所以它会占用一部分CPU资源，所以设计好垃圾回收器线程的数量

###  Jvm日志出现OOM问题，不能创建本地线程，分析原因，并给出解决办法 1 

机器内存不足，可以减少堆内存大小，或者增加机器内存大小

###  方法信息存储区域 1 

方法区

### 简述一下JMM 1

每个线程有一个高速缓存，所有线程共享一个共享内存。
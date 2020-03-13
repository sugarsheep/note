## 说明

## 目录

## jvm概述

### jvm组成结构

![1563374405509](images\1563374405509.png)

![1563374529648](images\1563374529648.png)

> - java线程start之后，不会立刻执行，会等待操作系统进行调度才会真正执行
>
> - 线程状态可通过Thread.State枚举类进行查看，该枚举类有6个值
>
>   - NEW：新建状态，即就绪状态
>   - RUNNABLE：运行状态
>   - BLOCKED：阻塞状态
>   - WAITING：等待状态
>   - TIMED_WAITING：等待状态
>   - TERMINATED：终止状态
>
>   其中WAITING、TIMED_WAITING组合起来为等待状态
>
> - 异常抛出是从运行时数据区进行抛出

### 类加载器和JNI(java本地库接口)

![1563376283918](images\1563376283918.png)

> - bootstrap加载器：根加载器，由c++语言编写，在java代码中打印出来是null，它负责加载rt.jar,即运行时依赖，我们常用的类都在里面
> - ext：扩展类加载器，java语言写的，加载jre/lib/ext/*.jar
> - App:应用类加载器，加载我们自己编写的类
> - 自定义类加载器：

### 类加载器的双亲委派机制

> - 当需要加载一个类时，会按根加载器--》ext加载器--》app加载器，逐层向下查找，没找到就报ClassNotFoundException
> - 每个类加载器都有一个指向父加载器的成员属性，当需要当前类加载器加载类时，它会先委派其父类加载器先加载，父加载器无法加载，再自己进行加载
> - 若自定义一个java.lang.String类，执行其main方法就会报错，因为根加载器会加载到jdk的String类，该类没有main方法，就会包找不到main方法的错误
> - 双亲委派机制的一个作用：保证自定义的类不会污染jdk自带的类，因为就算定义了相同的类，根据双亲委派机制，也会先加载jdk的，就不会造成程序错误，否则源代码之前引用的类就错乱了

### 方法区、程序计数器、本地方法栈

![1583596904956](images/1583596904956.png)

![1563376495381](images\1563376495381.png)

> - jvm的优化99%是优化堆，1%是优化方法区
> - 方法区主要放的是构造函数、接口定义
> - 本地方法主要是一些系统函数和其它语言库（如c语言）

#### 方法区

> - 线程共享
> - 存在gc垃圾回收
> - 方法区不是存放方法的
> - 方法区是供各线程共享的运行时内存区域，**它存储了每一个类的结构信息**（类加载器将class文件加载到jvm里形成对应的Class类），例如运行时常量池、字段和方法数据、构造函数和普通方法的字节码内容。这是虚拟机的规范，在不同的虚拟机里实现方式是不一样的，最典型的就是永久代（PermGen sapce，1.7）和元空间（Metaspace,1.8）
> - **实例变量存在堆内存中，和方法区无关**

### 栈、堆

> 栈管运行，堆管存储
>
> 栈的例子：子弹弹夹
>
> 队列的例子：排队买票

#### 栈

**栈是什么**

> - 栈也叫栈内存，主管java程序的运行，是在线程创建时创建，它的生命周期是跟随线程的生命周期，线程结束栈内存也就释放
> - 对于栈来说不存在垃圾回收，只要线程结束栈就over
> - 生命周期和线程一致，是线程私有的。
> - 基本类型的变量和对象的引用都是在函数的栈内存分配

**栈帧**

> - 一个方法调用就会形成一个栈帧，调用方法时压栈，方法结束时出栈
> - 栈帧中主要保存3类数据
>   - 本地变量(Local Variable)：输入参数、输出参数以及方法内的局部变量
>   - 栈操作（Operand Stack）：记录出栈、入栈的操作，即方法入口和出口
>   - 栈帧数据（Frame Data）:包括类文件、方法等
> - 每个方法执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息，每个方法从调用到执行完毕的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。栈的大小与具体的jvm实现有关，通常在256k~756k之间，约等于1Mb左右

**StackOverFlowError**

> - 栈空间溢出错误，它是一个Error，在如递归调用时无法终止的情况下回产生该错误

#### 栈的运行原理

![1563455409540](images\1563455409540.png)

![1563455584040](images\1563455584040.png)

#### 堆、栈、方法区的关系

> - HotSpot是使用指针的方式来访问对象：
> - Java堆中会存放访问**类元数据**的地址，
> - reference存储的就直接是对象的地址
>
> ![1583652039310](images/1583652039310.png)

#### 1.7堆内存示意图

> - 一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行
> - 刚创建的对象存在于伊甸区中，当伊甸区快满时，开始进行垃圾回收
> - 堆逻辑上分为3个部分：新生代+老年代+永久代（元空间）
> - 1.8时使用元空间替换了永久代 

![1563457387978](images\1563457387978.png)

##### 新生区

![1563457822821](images\1563457822821.png)

> - -Xms参数说明：-X是固定写法，m是单位mb,s是start，而-Xmx的x是max
> - 默认情况下jvm的内存为物理内存的1/4

##### 年老区

> - 一般经过15次垃圾回收之后还存活的对象会被放入年老区

![1563459251088](images\1563459251088.png)

##### 永久区

> 永久代中不存在垃圾回收，就算内存用完了也不会

![1563459501174](images\1563459501174.png)

##### 程序内存划分小结

![1563460629827](images\1563460629827.png)



> - JVM从软件层面屏蔽了底层硬件、指令层面的细节
> - JVM运行时数据区，分为2部分：数据、指令
>   - 数据：方法区、堆
>   - 指令：程序计数器、虚拟机栈、本地方法栈
> - 程序计数器：指向当前线程正在执行的字节码指令的地址（行号）,主要在线程切换时保存状态
> - 虚拟机栈：存储当前线程运行方法时所需要的方法参数、指令、返回地址
> - 栈帧：局部变量表、操作数栈、动态链接、出口
> - javap -v class:反编译class文件
> - 动态链接：

## **MinorGC**的过程

### 新生代和老年代内存比例图

![1583760842053](images/1583760842053.png)

### MinorGC（young gc）的过程

> 复制->清空->互换

#### eden、SurvivorFrom 复制到 SurvivorTo，年龄+1 

> - 首先，当Eden区满的时候会触发第一次GC,把还活着的对象拷贝到SurvivorFrom区，
> - 当Eden区再次触发GC的时候会扫描Eden区和From区域,对这两个区域进行垃圾回收，经过这次回收后还存活的对象,则直接复制到To区域（如果有对象的年龄已经达到了老年的标准，则复制到老年代区），同时把这些对象的年龄+1

#### 清空 eden、SurvivorFrom 

> - 然后，清空Eden和SurvivorFrom中的对象，也即复制之后有交换，谁空谁是to

#### SurvivorTo和 SurvivorFrom 互换 

> - 最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区。
>
> - 部分对象会在From和To区域中复制来复制去,如此交换15次(由JVM参数MaxTenuringThreshold决定,这个参数默认是15),最终如果还是存活,就存入到老年代

## 方法区

![1583762113777](images/1583762113777.png)

> - 实际而言，方法区（Method Area）和堆一样，是各个线程共享的内存区域，它用于存储虚拟机加载的：类信息+普通常量+静态常量+编译器编译后的代码等等，虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一个别名叫做Non-Heap(非堆)，目的就是要和堆分开。
> - 对于HotSpot虚拟机，很多开发者习惯将方法区称之为“永久代(Parmanent Gen)” ，但严格本质上说两者不同，或者说使用永久代来实现方法区而已，永久代是方法区(相当于是一个接口interface)的一个实现，
> - jdk1.7的版本中，已经将原本放在永久代的字符串常量池移走。
> - jdk1.7永久存储区是一个常驻内存区域，用于存放JDK自身所携带的Class,Interface 的元数据，也就是说它存储的是运行环境必须的类信息，被装载进此区域的数据是不会被垃圾回收器回收掉的，关闭JVM 才会释放此区域所占用的内存

## 堆参数调优

### **jdk1.7堆内存结构图**

> - -Xms:-X是固定写法，m是M的意思，s表示start，即堆初始化内存大小
> - -Xmx:堆最大内存
> - -Xmn:新生代和老年代的比例，一般不调整，默认为1:2
> - -XX:PermSize:永久代的初始化内存大小
> - -XX:MaxPermSize:永久代最大内存大小

![1583762260904](images/1583762260904.png)

### **jdk1.8堆内存结构图**

> - JDK 1.8之后将最初的永久代取消了，由元空间取代
> - 元空间与永久代之间最大的区别在于：永久带使用的JVM的堆内存，但是java8以后的**元空间并不在虚拟机中而是使用本机物理内存**。
> - 因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入native memory, 字符串池和类的静态变量放入java 堆中，这样加载多少类的元数据就不再由MaxPermSize 控制, 而由系统的实际可用空间来控制

![1583762301079](images/1583762301079.png)

### 参数调优

![1583763775343](images/1583763775343.png)

> - java将运行时环境封装成了Runtimr对象，通过它可以获取运行时的各种参数信息
> - 最好将-Xms和-Xmx设置成一样，避免内存忽高忽低

#### 获取系统可用cpu核心数

```java
Runtime.getRuntime().availableProcessors()
```

#### 获取堆初始化大小和最大大小

```java
System.out.println(Runtime.getRuntime().totalMemory());
System.out.println(Runtime.getRuntime().maxMemory());
//默认为1/64系统内存
System.out.println("-Xms:"+(Runtime.getRuntime().totalMemory()/1024/1024)+"MB");
//默认为1/4系统内存
System.out.println("-Xmx:"+(Runtime.getRuntime().maxMemory()/1024/1024)+"MB");
```

#### idea jvm参数设置

![1583764463964](images/1583764463964.png)

**运行结果**

```
12
1029177344
1029177344
-Xms:981MB
-Xmx:981MB
Heap
 PSYoungGen      total 305664K, used 20971K [0x00000000eab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 262144K, 8% used [0x00000000eab00000,0x00000000ebf7afb8,0x00000000fab00000)
  from space 43520K, 0% used [0x00000000fd580000,0x00000000fd580000,0x0000000100000000)
  to   space 43520K, 0% used [0x00000000fab00000,0x00000000fab00000,0x00000000fd580000)
 ParOldGen       total 699392K, used 0K [0x00000000c0000000, 0x00000000eab00000, 0x00000000eab00000)
  object space 699392K, 0% used [0x00000000c0000000,0x00000000c0000000,0x00000000eab00000)
 Metaspace       used 3095K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 336K, capacity 388K, committed 512K, reserved 1048576K
```

**结果分析**

> - jvm堆由PSYoungGen（年轻代）和ParOldGen（老年代）组成，它们的大小305664K+699392K正好是981MB，这也证明了元空间确实不在堆中

#### OOM

```java
public class Test01 {

    public static void main(String[] args) {
        String str = "hello world";
        Random random = new Random();
        while (true) {
            str = str + str + random.nextInt(888888888) + random.nextInt(999999999);
        }
    }
}
```

**运行结果**

> - 程序先进行young gc，之后达到full gc条件进行full gc，之后内存还是不够则抛出`java.lang.OutOfMemoryError: Java heap space`

```
[GC (Allocation Failure) [PSYoungGen: 512K->504K(1024K)] 512K->532K(1536K), 0.0004671 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1016K->488K(1024K)] 1044K->636K(1536K), 0.0006986 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1000K->488K(1024K)] 1148K->740K(1536K), 0.0004536 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 959K->491K(1024K)] 1211K->884K(1536K), 0.0003819 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 491K->468K(1024K)] [ParOldGen: 392K->280K(512K)] 884K->748K(1536K), [Metaspace: 3061K->3061K(1056768K)], 0.0033904 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) --[PSYoungGen: 954K->954K(1024K)] 1234K->1426K(1536K), 0.0005113 secs] [Times: user=0.03 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 954K->0K(1024K)] [ParOldGen: 472K->414K(512K)] 1426K->414K(1536K), [Metaspace: 3073K->3073K(1056768K)], 0.0033759 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 486K->348K(1024K)] [ParOldGen: 414K->356K(512K)] 900K->704K(1536K), [Metaspace: 3074K->3074K(1056768K)], 0.0036726 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 348K->348K(1024K)] [ParOldGen: 356K->344K(512K)] 704K->692K(1536K), [Metaspace: 3074K->3074K(1056768K)], 0.0034826 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.sugar.Test01.main(Test01.java:11)
Heap
 PSYoungGen      total 1024K, used 416K [0x00000000ffe80000, 0x0000000100000000, 0x0000000100000000)
  eden space 512K, 81% used [0x00000000ffe80000,0x00000000ffee8270,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 45% used [0x00000000fff00000,0x00000000fff3ac20,0x00000000fff80000)
 ParOldGen       total 512K, used 344K [0x00000000ffe00000, 0x00000000ffe80000, 0x00000000ffe80000)
  object space 512K, 67% used [0x00000000ffe00000,0x00000000ffe56238,0x00000000ffe80000)
 Metaspace       used 3156K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 342K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 1

```

#### gc日志解读

![1583766058138](images/1583766058138.png)

**young gc**

![1583765713854](images/1583765713854.png)

**full gc**

```
[Full GC (Allocation Failure):gc类型
[PSYoungGen: 348K->348K(1024K)] ：新生代清理结果
[ParOldGen: 356K->344K(512K)] ：老年代清理结果
704K->692K(1536K),：堆内存
[Metaspace: 3074K->3074K(1056768K)], ：元空间
0.0034826 secs] ：gc耗时
[Times: user=0.00 sys=0.00, real=0.00 secs] ：各方面耗时

```

## gc算法

> - jvm进行gc时，并非每次都对上面3个内存区域一起回收，大部分时候回收的都是指新生代。因此gc按照回收的区域又分为3种类型，一种是普通gc（Minor gc）,一种是全局gc（Major gc or Full gc）
> - **Minor gc和Full gc的区别**
>   - Minor gc：只针对新生代区域的gc,指发生在新生代的垃圾收集动作，因为大多数java对象存活率都不高，所以Minor gc非常频繁，一般回收速度也比较块
>   - Full gc：指发生在老年代的垃圾收集动作，出现了Full gc，经常会伴随至少一次的Minor gc（但并不是绝对的）。Full gc的速度一般要比Minor gc慢上10倍以上，因为Full gc需要清理的内存更多

### System.gc()

> - 手动开启gc，但并不会立刻执行

### 引用计数法

![1583767315567](images/1583767315567.png)

### 复制算法

> - 年轻代中使用的是minor gc,这种gc算法采用的是复制算法
>
> - HotSpot JVM把年轻代分为了3个部分，1个Eden区和2个Survivor区(分别叫from和to)。默认比例为8：1:1，一般情况下，新创建的对象会被分配到Eden区（一些大对象特殊处理），这些对象经过第一次Minor GC后，如果仍然存活，将会被移动到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度（默认15次）时，就会被移动到年老代中。因为年轻代中的对象基本上都是朝生夕死的（90%以上），所以在年轻代使用的垃圾回收算法是复制算法，复制算法的基本思想就是将内存分为2块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片
>
> - ![1583942405012](images/1583942405012.png)
>
> - 在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区, Survivor区“To”是空的。紧接着进行GC， Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。
>
>   ![1584019583690](images/1584019583690.png)
>
> - 因为Eden区对象一般存活率较低，一般的，使用两块10%的内存作为空闲和活动区间，而另外80%的内存，则是用来给新建对象分配内存的。一旦发生GC，将10%的from活动区间与另外80%中存活的eden对象转移到10%的to空闲区间，接下来，将之前90%的内存全部释放，以此类推。

#### 复制算法的缺点

> - 复制算法它的缺点也是相当明显的。
> - 1、它浪费了一半的内存，这太要命了。
> - 2、如果对象的存活率很高，我们可以极端一点，假设是100%存活，那么我们需要将所有对象都复制一遍，并将所有引用地址重置一遍。复制这一工作所花费的时间，在对象存活率达到一定程度时，将会变的不可忽视。所以从以上描述不难看出，复制算法要想使用，最起码对象的存活率要非常低才行，而且最重要的是，我们必须要克服50%内存的浪费。

### 标记清除

> - 老年代一般是由标记清除或者是标记清除与标记整理的混合实现
>
> - 算法分成标记和清除两个段，先标记出要回收的对象，然后统一回收这些对象。形如:
>
>   ![1584021221651](images/1584021221651.png)
>
> - 标记清除法虽然解决了内存占用的问题，但是会扫描2次（标记时1次+回收时一次）耗时严重，而且会产生内存碎片，内存不连续
>
> - 用通俗的话解释一下标记清除算法，就是当程序运行期间，若可以使用的内存被耗尽的时候，GC线程就会被触发并将程序暂停，随后将要回收的对象标记一遍，最终统一回收这些对象，完成标记清理工作接下来便让应用程序恢复运行。

#### 标记清除法缺点

> - 1、首先，它的缺点就是效率比较低(递归与全堆对象遍历)，而且在进行GC的时候，需要停止应用程序，这会导致用户体验非常差劲
> - 2、其次，主要的缺点则是这种方式清理出来的空闲内存是不连续的，这点不难理解，我们的死亡对象都是随即的出现在内存的各个角落的，现在把它们清除之后，内存的布局自然会乱七八糟。而为了应付这一点，JVM就不得不维持一个内存的空闲列表，这又是一种开销。而且在分配数组对象的时候，寻找连续的内存空间会不太好找。

### 标记压缩

> - 老年代一般是由标记清除或者是标记清除与标记整理的混合实现
> - ![1584022210459](images/1584022210459.png)
> - 在整理压缩阶段，不再对标记的对像做回收，而是通过所有在活对像都向一端移动，然后直接清除边界以外的内存。可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。
> - 标记/整理算法唯一的缺点就是效率也不高，不仅要标记所有存活对象，还要整理所有存活对象的引用地址。从效率上来说，标记/整理算法要低于复制算法。

### 4大GC算法对比

> - 内存效率：复制算法>标记清除算法>标记整理算法（此处的效率只是简单的对比时间复杂度，实际情况不一定如此）。
> - 内存整齐度：复制算法=标记整理算法>标记清除算法。
> - 内存利用率:标记整理算法=标记清除算法>复制算法。
> - 可以看出，效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存，而为了尽量兼顾上面所提到的三个指标，标记/整理算法
> - 相对来说更平滑一些，但效率上依然不尽如人意，它比复制算法多了一个标记的阶段，又比标记/清除多了一个整理内存的过程

### 分代收集算法

> - 次数上频繁收取young区
> - 次数上较少手机old区
> - 基本不动元空间
> - 对于年轻代，年轻代特点是区域相对老年代较小，对像存活率低。这种情况复制算法的回收整理，速度是最快的。复制算法的效率只和当前存活对像大小有关，因而很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过hotspot中的两个survivor的设计得到缓解。
> - 对于老年代，老年代的特点是区域较大，对像存活率高。这种情况，存在大量存活率高的对像，复制算法明显变得不合适。一般是由标记清除或者是标记清除与标记整理的混合实现。
>   - Mark阶段的开销与存活对像的数量成正比，这点上说来，对于老年代，标记清除或者标记整理有一些不符，但可以通过多核/线程利用，对并发、并行的形式提标记效率。
>   - Sweep阶段的开销与所管理区域的大小形正相关，但Sweep“就地处决”的特点，回收的过程没有对像的移动。使其相对其它有对像移动步骤的回收算法，仍然是效率最好的。但是需要解决内存碎片问题。
>   - Compact阶段的开销与存活对像的数据成开比，如上一条所描述，对于大量对像的移动是很大开销的，做为老年代的第一选择并不合适。
>   - 基于上面的考虑，老年代一般是由标记清除或者是标记清除与标记整理的混合实现。以hotspot中的CMS回收器为例，CMS是基于Mark-Sweep实现的，对于对像的回收效率很高，而对于碎片问题，CMS采用基于Mark-Compact算法的Serial 0ld回收器做为补偿措施：当内存回收不佳(碎片导致的Concurrent Mode Failure时)，将采用Serial Old执行Full GC以达到对老年代内存的整理。

## JMM（java内存模型）

> - 由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝到的线程自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的变量副本拷贝，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成，其简要访问过程如下图:
> - ![1584024216244](images/1584024216244.png)
> - 1

### JMM内存可见性验证代码

> - 以下代码主线程由于始终读取自己工作内存的副本，所以无法退出循环，若给变量a使用volatile声明即可保证可见性

```java
class MyNumber {
    private int a = 0;

    public int getA() {
        return a;
    }

    public void change() {
        a = 100;
    }
}

public class Test01 {

    public static void main(String[] args) {
        MyNumber myNumber = new MyNumber();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "--------------come in");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            myNumber.change();
            System.out.println(Thread.currentThread().getName() + "------" + myNumber.getA());
        }, "AA").start();

        while (myNumber.getA() == 0) {

        }

        System.out.println(Thread.currentThread().getName() + "-------main");
    }
}
```


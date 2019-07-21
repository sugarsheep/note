## 说明

## 目录

## jvm概述

### jvm组成结构

![1563374405509](E:\git\note\java\jvm\images\1563374405509.png)

![1563374529648](E:\git\note\java\jvm\images\1563374529648.png)

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

![1563376283918](E:\git\note\java\jvm\images\1563376283918.png)

### 方法区、程序计数器、本地方法栈

![1563376495381](E:\git\note\java\jvm\images\1563376495381.png)

> - jvm的优化99%是优化堆，1%是优化方法区
> - 方法区主要放的是构造函数、接口定义

### 栈、堆

> 栈管运行，堆管存储

#### 栈

![1563454755340](E:\git\note\java\jvm\images\1563454755340.png)

#### 栈的运行原理

![1563455409540](E:\git\note\java\jvm\images\1563455409540.png)

#### 栈帧之间的关系

![1563455584040](E:\git\note\java\jvm\images\1563455584040.png)

#### 堆内存示意图

![1563457387978](E:\git\note\java\jvm\images\1563457387978.png)

> - 刚创建的对象存在于伊甸区中，当伊甸区快满时，开始进行垃圾回收

##### 新生区

![1563457822821](E:\git\note\java\jvm\images\1563457822821.png)

> - -Xms参数说明：-X是固定写法，m是单位mb,s是start，而-Xmx的x是max
> - 默认情况下jvm的内存为物理内存的1/4

##### 年老区

> - 一般经过15次垃圾回收之后还存活的对象会被放入年老区

![1563459251088](E:\git\note\java\jvm\images\1563459251088.png)

##### 永久区

> 永久代中不存在垃圾回收，就算内存用完了也不会

![1563459501174](E:\git\note\java\jvm\images\1563459501174.png)

##### 程序内存划分小结

![1563460629827](E:\git\note\java\jvm\images\1563460629827.png)
# CopyOnWriteArrayList

## 描述

> - CopyOnWriteArrayList是线程安全的list，采用**写时复制**技术实现线程安全，所谓写时复制，就集合在添加元素时，会复制一个新的数组进行操作，其它线程读取的是原始集合的数据，添加操作完成后，在将原始数组引用覆盖

## 原理

> - 写时复制+ReentrantLock+volatile
> - 写时复制：集合添加元素是复制一个新数组进行操作
> - ReentrantLock：保证多线程并发添加时的线程安全
> - volatile：保证多线程时每次读取到的数据都是最新的，volatile会强制线程去读取主内存最新的数据

## 源码解读

> - 同ArrayList，都是使用**Object数组**的方式存储元素,volatile修饰是为了保证其在多线程环境下的可见性，即都会读取最新的数组中的元素
>
>   ```java
>   private transient volatile Object[] array;
>   ```
>
> - 构造方法，默认构造方法会创建一个**空数组**
>
> - 添加元素
>
>   ```java
>   public boolean add(E e) {
>       final ReentrantLock lock = this.lock;
>       //加锁
>       lock.lock();
>       try {
>           Object[] elements = getArray();
>           int len = elements.length;
>           //复制新数组，新数组长度=原数组长度+1
>           Object[] newElements = Arrays.copyOf(elements, len + 1);
>           //新数组最后一个元素设置为新添加的元素
>           newElements[len] = e;
>           //修改引用，将引用指向新的数组
>           setArray(newElements);
>           return true;
>       } finally {
>           //解锁
>           lock.unlock();
>       }
>   }
>   ```
>
> - 总的来说，凡是和修改相关的方法（add,addAll,remove等），都会加锁，并使用Arrays.copyOf复制数组进行操作
>
> - 支持序列化，提供了resetLock方法在反序列化后重置锁
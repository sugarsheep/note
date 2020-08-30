## 说明

## 目录

## 实现接口

### RunnableFuture

> - RunnableFuture是一个组合接口，它既有Runnable的功能，又有Future的功能
>   - Runnable：定义线程可执行的任务
>   - Future：执行的任务可以获取结果

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    //执行任务
    void run();
}
```

### Future

> 执行的任务可以获取结果

```java
public interface Future<V> {
    //取消任务
    boolean cancel(boolean mayInterruptIfRunning);

    //返回任务是否已经取消
    boolean isCancelled();

    //任务是否完成
    boolean isDone();

    //阻塞获取任务执行结果
    V get() throws InterruptedException, ExecutionException;

    //阻塞获取任务执行结果，带有超时时间
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

## 属性

> 状态state的转换规则
>
> ```
> * NEW -> COMPLETING -> NORMAL
> * NEW -> COMPLETING -> EXCEPTIONAL
> * NEW -> CANCELLED
> * NEW -> INTERRUPTING -> INTERRUPTED
> ```

```java
//任务状态
private volatile int state;
//任务处于新建状态，还没有开始运行
private static final int NEW          = 0;
//任务处于完成中状态，还没有完成
private static final int COMPLETING   = 1;
//任务已完成
private static final int NORMAL       = 2;
//任务出现异常
private static final int EXCEPTIONAL  = 3;
//任务被取消
private static final int CANCELLED    = 4;
//任务正在被中断
private static final int INTERRUPTING = 5;
//任务处于中断状态
private static final int INTERRUPTED  = 6;

//实际的执行任务，Runnable也会被包装成Callable
private Callable<V> callable;
//任务结果：若任务正常结束，为返回结果；任务抛出异常，则是异常对象
private Object outcome; 
//执行任务的线程
private volatile Thread runner;
//可能有多个线程在调用get方法获取结果，但是任务若没有执行完成，需要对调用线程进行阻塞
private volatile WaitNode waiters;


static final class WaitNode {
    //被阻塞的线程
    volatile Thread thread;
    //下一个阻塞线程
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```

## 构造方法

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    //设置任务为新建状态
    this.state = NEW;   
}

public FutureTask(Runnable runnable, V result) {
    //将Runnable包装成Callable，并使用传入的result作为返回值
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;     
}
```

Executors.callable

> 使用RunnableAdapter适配器将Runnable包装成Callable

```java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}


    static final class RunnableAdapter<T> implements Callable<T> {
        final Runnable task;
        final T result;
        RunnableAdapter(Runnable task, T result) {
            this.task = task;
            this.result = result;
        }
        public T call() {
            task.run();
            return result;
        }
    }
```

## run()

```java
public void run() {
    //1.state必须为新建状态，因为任务可能被外部线程取消
    //2.设置当前任务被持有的线程runner：若设置成功，则表示当前任务没有被其它线程持有；若设置失败，则当前任务已被其它线程持有执行
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        //外部传入的callable可能为null；任务也可能被取消
        if (c != null && state == NEW) {
            //任务返回结果
            V result;
            //任务执行成功的标志
            boolean ran;
            try {
                //调用任务逻辑，获取任务结果
                result = c.call();
                //任务执行完成，修改标志
                ran = true;
            } catch (Throwable ex) {
                //任务异常，result设置为null
                result = null;
                //设置标志位失败
                ran = false;
                //设置异常对象
                setException(ex);
            }
            //任务执行成功，则设置结果
            if (ran)
                //设置任务结果
                set(result);
        }
    } finally {
        //将任务持有线程重置为null
        runner = null;
        int s = state;
        //检查任务中断
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

## setException

```java
protected void setException(Throwable t) {
    //cas设置任务状态从NEW--》COMPLETING
    //可能失败，外部线程取消任务
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        //设置outcome为异常对象
        outcome = t;
        //设置任务状态为EXCEPTIONAL
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        //处理阻塞到get方法的线程
        finishCompletion();
    }
}
```

## set

```java
protected void set(V v) {
    //cas设置任务状态从NEW--》COMPLETING
    //可能失败，外部线程取消任务
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        //设置outcome为任务结果
        outcome = v;
        //设置任务状态为NORMAL，任务成功执行
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        //处理阻塞到get方法的线程
        finishCompletion();
    }
}
```

## get

```java
//可能有多个线程get获取任务结果
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    //条件成立：任务处于新建或执行中，就等待任务执行结束，调用get的外部线程会在这里被阻塞
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

## awaitDone

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    //0：不带超时
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    //当前线程会被封装成WaitNode对象
    WaitNode q = null;
    //当前线程封装成WaitNode的对象又没有入队
    boolean queued = false;
    //自旋
    for (;;) {
        //线程被其它线程中断，重置当前线程中断标志
        if (Thread.interrupted()) {
            //从队列移除当前线程的节点
            removeWaiter(q);
            throw new InterruptedException();
        }

        //任务被其它线程unpark(thread)唤醒或正常逻辑
        int s = state;
        //任务处于完成，中断或取消状态
        if (s > COMPLETING) {
            //此时执行任务的线程任务已执行完成，可以释放
            if (q != null)
                q.thread = null;
            return s;
        }
        //任务快要完成了，释放一下cup，等待任务完成
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        //第一次循环，创建WaitNode对象
        else if (q == null)
            q = new WaitNode();
        //第二次循环，当前线程已创建WaitNode对象，但还没有入队,会自旋到成功入队为止
        else if (!queued)
            //q.next = waiters：将当前WaitNode指向当前队列的头结点waiters，waiters一直指向队列头结点
            //cas修改原始的waiters，使其指向新的头结点，类似栈顶
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        //第三次循环，判断超时
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            //不带超时直接阻塞当前线程（处于waiting状态），需要其它线程将其唤醒或中断
            LockSupport.park(this);
    }
}
```

## report

```java
private V report(int s) throws ExecutionException {
    //正确情况下：outcome为任务执行结果
    //异常情况下：outcome为任务抛出的异常对象
    Object x = outcome;
    //任务正常执行结束，直接返回结果
    if (s == NORMAL)
        return (V)x;
    //任务被取消，抛出异常
    if (s >= CANCELLED)
        throw new CancellationException();
    //任务抛出异常
    throw new ExecutionException((Throwable)x);
}
```

## finishCompletion

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    //q执行waiters链表的头结点，将链表中的节点对应的线程依次进行唤醒
    for (WaitNode q; (q = waiters) != null;) {
        //cas设置当前头结点为空，外部线程调用cancel也会进入finishCompletion，
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                //获取当前节点的线程
                Thread t = q.thread;
                if (t != null) {
                    //help gc
                    q.thread = null;
                    //线程不为空进行唤醒
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                //next == null说明已到达链表尾部，可以结束循环
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                //指向下一个节点继续循环
                q = next;
            }
            break;
        }
    }

    //扩展点
    done();

    callable = null;        // to reduce footprint
}
```

## removeWaiter

```java
//节点对应的线程被中断或超时
private void removeWaiter(WaitNode node) {
    //节点不为空，则从链表移除
    if (node != null) {
        //help gc
        node.thread = null;
        retry:
        for (;;) {
            //pred:前置节点，q：当前节点，s:下一个节点
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                //s指向下一个节点
                s = q.next;
                //当前节点是中间的节点，线程不为空，则设置其为前置节点
                if (q.thread != null)
                    pred = q;
                else if (pred != null) {
                    //前置节点直接指向s，断开到当前节点的链接
                    pred.next = s;
                    //前置节点可能被其它线程出队了，再次进行校验
                    if (pred.thread == null) // check for race
                        //进行重试
                        continue retry;
                }
                //当前节点是头结点，上面2个if都会跳过，则使用下一个节点作为链表新的头结点waiters
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                      q, s))
                    continue retry;
            }
            break;
        }
    }
}
```

## cancel

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    //条件1：任务处于NEW状态才可以取消
    //条件2：根据参数mayInterruptIfRunning cas设置任务状态成功
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {
        // 中断线程
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                //任务在线程池还没有被执行，则为null
                if (t != null)
                    //设置线程中断标志
                    t.interrupt();
            } finally {
                // 设置任务尾中断为状态
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        //唤醒被get阻塞的线程
        finishCompletion();
    }
    return true;
}
```
# 面试

## 一、多线程

### 1、多线程的目的是什么？

充分利用cpu资源，并发做多件事

### 2、单核cpu机器上和不适合用多线程？

适合，如果是单线程，线程中需要等待IO时，此时CPU就空闲出来了。

### 3、线程什么时候会让出cpu？

1）阻塞、wait、await、等待io

2）sleep

3）yield

4）结束了

### 4、线程的本质是什么？

1）一条代码执行流，完成一组代码的执行

2）这一组代码，我们往往称为一个任务。

### 5、cpu做的是什么工作？

1）执行代码

2）代码-线程-cpu

### 6、线程是不是越多越好？

1）线程在java中是一个对象，每个java线程都需要一个操作系统线程支持。线程创建、销毁需要时间。如果创建时间 + 销毁时间 > 执行任务时间 就很不划算。

2）java对象占用堆内存，操作系统线程占用系统内存，根据jvm规范，一个线程默认最大栈大小1M，这个栈空间是需要系统内存中分配的。线程过多，会消耗很多内存。

3）操作系统需要频繁切换线程上下文，影响性能。

### 7、该如何正确使用多线程？

1）多线程目的：充分利用cpu并发做事

2）线程的本质：将代码送给cpu执行

3）用合适数量的卡车不断运送代码即可

4）这合适数量的线程就构成了一个池

## 二、线程池

### 1、线程池

#### 1）3个常用方式

~~~java
Executors.newFixedThreadPool(int) // 执行长期的任务，固定线程
Executors.newSingleThreadExecutor() // 一个任务一个任务，就一个线程
Executors.newCachedThreadPool() // 适用执行很多短期任务，n个线程   
~~~

**上述3个在工作中一个都不用****，自定义的线程池**

#### 2）7个参数

~~~java
1.corePoolSize：线程池中的常驻核心线程数
2.maximumPoolSize：线程池能够容纳同时执行的最大线程数，此值必须大于等于1
3.keepAliveTime：多余的空闲线程的存活时间。当前线程池数量超过corePoolSize时，且空闲时间达到keepAliveTime值时，多余空闲线程就会别销毁知道只剩下corePoolSize线程为止。
4.unit：keepAliveTime的单位
5.workQueue：任务队列，被提交但尚未被执行的任务
6.threadFactory：生成线程池中工作线程的线程工厂，默认即可
7.handler：拒绝策略，表示当队列满了并且工作线程大于等于线程池的最大线程数
~~~

#### 3）工作原理

~~~txt
1.在创建线程池后，等待提交过来的任务请求
2.当调用execute()方法添加一个请求任务，线程池做如下判断：
1）如果正在运行的线程数量小于corePoolSize，马上启动线程运行这个任务。
2）如果正在运行的线程数量大于或等于corePoolSize，将这个任务放入队列
3）如果这时候队列满了且运行的线程数量还小于maximumPoolSize，创建非核心线程立即运行该任务
4）如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，线程池启动拒绝策略
3.当一个线程完成任务时，会总队列中取下一个任务来执行
4.当一个线程无事可做超过一定时间（keepAliveTime）时，线程池判断：
1）若当前运行的线程数大于corePoolSize，这个线程会被停止。
2）所有线程池的所有任务完成后，最终会收缩到corePoolSize的大小。
~~~

#### 4）拒绝策略

AbortPolicy（默认）：直接抛出RejectedExecutionExecption异常阻止系统正常运行

CallerRunsPolicy：有该调用者执行该任务

DiscardOldestPolicy：抛弃队列中等待最久的任务，然后将当前任务加入队列中尝试再次提交

DiscardPolicy：直接丢弃任务。



1）接收任务，放入仓库

2）工作线程从仓库取任务，执行

3）当没有任务时，线程阻塞，当有任务时唤醒线程执行

### 2、任务用什么表示？

1）Runnable

2）Callable：对Runnable的改进，可以有返回值，可以抛出异常

~~~java
Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
        Thread.sleep(1000);
        Random random = new Random();
        return "返回值：" + random.nextInt(100);
    }
};

FutureTask<String> futureTask = new FutureTask<String>(callable);
new Thread(futureTask).start();
// 等待线程执行完成
String result = futureTask.get();
System.out.println(result);

// 线程池
ThreadPoolExecutor pool = new ThreadPoolExecutor(10, 20, 60, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
Future<String> future =  pool.submit(callable);
String rs = future.get();
System.out.println(rs);
pool.shutdown();
~~~

### 3、队列是什么？

#### 1）BlockingQueue阻塞队列，线程安全的

在队列为空时的获取阻塞，在队列满时放入阻塞。

BlockingQueue方法以四种形式出现，对于不能立即满足但可能在将来某一时刻可以满足的操作，这四种形式的处理方式不同：第一种是抛出异常，第二种是返回一个特殊值（null或false，具体取决于操作），第三种是操作可以成功前，无限期的阻塞当前线程，第四种是在放弃前只在给定的最大时间限制内阻塞。如下表：

|      | 抛出异常  | 特殊值   | 阻塞   | 超时               |
| ---- | --------- | -------- | ------ | ------------------ |
| 插入 | add(e)    | offer(e) | put(e) | offer(e,time,unit) |
| 移除 | remove()  | poll()   | take() | poll(time,unit)    |
| 检查 | eiement() | peek()   | 不可用 | 不可用             |

**ArrayBlockingQueue：有数组结构组成的有界阻塞队列。**

**LinkedBlockingQueue：由链表结构组成的有界（默认值为Integer.MAX_VALUE）阻塞队列**

**SynchronousQueue：不存储元素的阻塞队列，即单个元素的队列（保存一个数据）**

PriorityBlockingQueue：支持优先级排序的无界阻塞队列

DelayQueue：使用优先级队列实现的延迟无界阻塞队列

LinkedTransferQueue：有链表结构组成的无界阻塞队列

LinkedTransferQueue：有链表结构组成的无界阻塞队列

LinkedBlocking**Deque**：有链表结构组成的**双向**阻塞队列

#### 2）生产者和消费者

##### 1、Synchronized

~~~java
/**
 * 生产者，消费者 synchronized,wait,notifyAll唤醒通知
 * 一个加1，一个减1
 * @version 
 * @author 傅泉明
 */
public class ProductConsumerObjectDemo {
    public static void main(String[] args) {
        ProductDataObject data = new ProductDataObject();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.add();
            }
        }, "product").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.delete();
            }
        }, "cunsumer").start();
        
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.add();
            }
        }, "product1").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.delete();
            }
        }, "cunsumer1").start();
    }
}
class ProductDataObject {
    volatile int count;
    public void add() {
        synchronized (this) {
            while (count == 1) {// 已生产，待消费,必须用while
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            count++;
            System.out.println(Thread.currentThread().getName() + ",count=" + count);
            notifyAll();
        }
    }
    public void delete() {
        synchronized (this) {
            while (count == 0) {// 已消费完，待生产,必须用while
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            count--;
            System.out.println(Thread.currentThread().getName() + ",count=" + count);
            notifyAll();
        }
    }
}
~~~

##### 2、Lock

~~~java
/**
 * 生产者，消费者 Lock,Condition.await,Condition.signalAll唤醒通知
 * @version 
 * @author 傅泉明
 */
public class ProductConsumerLockDemo {
    public static void main(String[] args) {
        ProductDataLock data = new ProductDataLock();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.add();
            }
        }, "product").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.delete();
            }
        }, "consumer").start();
        
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.add();
            }
        }, "product1").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data.delete();
            }
        }, "consumer1").start();
    }
}
class ProductDataLock {
    volatile int count;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    public void add() {
        lock.lock();
        try {
            while (count == 1) {// 已生产，待消费,必须用while
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            count++;
            System.out.println(Thread.currentThread().getName() + ",count=" + count);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
    public void delete() {
        lock.lock();
        try {
            while (count == 0) {// 已消费完，待生产,必须用while
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            count--;
            System.out.println(Thread.currentThread().getName() + ",count=" + count);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
~~~

##### 3、BlockQueue

~~~java
/**
 * 生产者，消费者 Queue，阻塞唤醒通知
 * 一个加1，一个减1
 * @version 
 * @author 傅泉明
 */
public class ProductConsumerQueueDemo {
    public static void main(String[] args) throws Exception {
        ProductDataQueue data = new ProductDataQueue();
        Thread t1 = new Thread(() -> {
            data.set();
        }, "product");t1.start();
        Thread t2 = new Thread(() -> {
            data.get();
        }, "consumer");t2.start();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        data.shutdown();
        t1.join();
        t2.join();
    }
}
class ProductDataQueue {
    private BlockingQueue<String> queue = new LinkedBlockingQueue<>(5);
    private volatile boolean running = true;// 是否运行
    // 消费线程
    private LinkedBlockingQueue<Thread> consumerThreadQueue = new LinkedBlockingQueue<>();
    public void get() {// 消费
        while (running) {
            try {
                System.out.println(Thread.currentThread().getName() + "，开始消费");
                consumerThreadQueue.offer(Thread.currentThread());
                String str = queue.take();
                System.out.println(Thread.currentThread().getName() + "，消费：" + str);
            } catch (InterruptedException e) {
//                e.printStackTrace();
            } finally {
                consumerThreadQueue.poll();
            }
        }
        System.out.println(Thread.currentThread().getName() + "，停止消费");
    }
    public void set() {// 生产
        while (running) {
            try {
                // 运行，阻塞
                String str = UUID.randomUUID().toString().substring(0,8); 
                queue.put(str);
                System.out.println(Thread.currentThread().getName() + "，生产：" + str);
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName() + "，停止生产");
    }
    public void shutdown() {
        this.running = false;
        // 获取阻塞的线程,当生产后休眠了1秒
        Thread t = null;
        while ((t = consumerThreadQueue.poll()) != null) {
//            System.out.println(t.getName() + "," + t.getState());
            t.interrupt();
        }
    }
}
~~~





#### 2）自定义线程池

~~~java
/**
 * 固定大小线程池demo
 * @version 
 * @author 傅泉明
 */
public class FixedSizeThreadPool {

    // 1.任务队列
    private BlockingQueue<Runnable> blockingQueue;
    // 2.线程集合
    private List<Thread> workers;
    // 3.每个线程执行任务-从队列中获取任务
    private static class Worker extends Thread {
        FixedSizeThreadPool pool;
        public Worker(FixedSizeThreadPool pool) {
            this.pool = pool;
        }
        @Override
        public void run() {
            while (pool.blockingQueue.isEmpty() == false || pool.isWorking == true) {
                try {
                    // 关闭线程池时不阻塞工作线程
                    Runnable tasker = null;
                    if (pool.isWorking == false) {
                        tasker = pool.blockingQueue.poll();
                    } else {
                        // 获取队列，获取不到阻塞工作线程
                        tasker = pool.blockingQueue.take();
                    }
                    if (tasker != null) {
                        tasker.run();                        
                        System.out.println("线程：" + Thread.currentThread().getName() + ",执行任务!");                        
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    // 4.初始化线程池
    public FixedSizeThreadPool(int poolSize, int taskerSize) {
        // 线程池大小，任务大小
        if (poolSize <= 0 || taskerSize <= 0) throw new IllegalArgumentException("非法参数");
        this.blockingQueue = new LinkedBlockingQueue<Runnable>(taskerSize);
        this.workers = new ArrayList<Thread>(poolSize);
        for (int i = 0; i < poolSize; i++) {
            Worker worker = new Worker(this);
            worker.start();
            workers.add(worker);
        }
    }
    // 5.将任务放入队列中
    public boolean execute(Runnable tasker) {
        if (isWorking) {
            return this.blockingQueue.offer(tasker);
        }
        return false;
    }
    // 6.关闭线程池
    // 1)不能添加新任务
    // 2)任务队列中任务要执行完成
    // 3)已阻塞线程，中断该线程
    private volatile boolean isWorking = true;// 标识是否运行中
    public void shutdown() {
        isWorking = false;
        for (Thread thread : workers) {
            if (Thread.State.BLOCKED.equals(thread.getState())) {
                thread.interrupt();// 中断线程
            }
        }
    }
    
    
    public static void main(String[] args) throws InterruptedException {
        FixedSizeThreadPool pool  = new FixedSizeThreadPool(3, 6);
        for (int i = 0; i < 10; i++) {
            boolean flag = pool.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("执行新任务：" + new Random().nextInt(100));
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });      
            if (flag == false) {
                i--;
                Thread.sleep(1000);
            }
        }
        pool.shutdown();
    }
}
~~~

### 4、如何确定合适数量的线程？

~~~java
// 获得CPU核数
Runtime.getRuntime().availableProcessors();
~~~

1）如果是计算型任务？

**CPU核数 + 1个线程的线程池**

2）如果是IO型任务？

- CPU核数 * 2

- **参考公式：CPU核数 / (1 - 阻塞系数)，阻塞系数在0.8-0.9之间**。**比如8和CPU：8 / (1 - 0.9) = 80个线程数**

## 三、Java命令

### 1、javap

javap是jdk自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（汇编指令）、本地变量表、异常表和代码行偏移量映射表、常量池等等信息。

~~~cmd
javap -v -p xxx.class
~~~

## 四、CAS

### 1、CAS（Compare and swap）

Compare and swap 比较和交换。属于硬件的同步原语，处理器提供基本内存操作的原子性保证。

CAS操作需要输入两个数值，一个旧值A（期望操作前的值）和一个新值B，在操作期间对旧值进行比较，若没有发生变化，才交换成新值，发生了变化则不较换。

Java中sun.misc.Unsafe类，体统了compareAndSwapInt()和compareAndSwapLong()等几个方法实现CAS。

实现AtomicInteger：

~~~java
/**
 * 实现AtomicInteger
 * @version 
 * @author 傅泉明
 */
public class CounterUnsafe {

    private volatile int count;
    private static long countOffset = 0;// 获取CounterUnsafe里count字段的偏移量
    private static Unsafe unsafe;
    static {
//        unsafe = Unsafe.getUnsafe();// 不能直接使用,JDK可以
        // 通过反射来实现
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);// 设置强制调用
            unsafe = (Unsafe)field.get(null);// 调用
            // 获取CounterUnsafe里count的偏移量
            countOffset = unsafe.objectFieldOffset(CounterUnsafe.class.getDeclaredField("count"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public void add() {
        for (;;) {
            int current = unsafe.getIntVolatile(this, countOffset);// 当前count的值
            if (unsafe.compareAndSwapInt(this, countOffset, current, current + 1)) break;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("123");
        final CounterUnsafe unsafe = new CounterUnsafe();
        for (int i = 0; i < 5; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 1000; j++)
                        unsafe.add();
                }
            }).start();
        }
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(unsafe.count);
    }
}
~~~

### 2、CAS缺点

1）循环时间长CPU开销大

2）只能保证一个共享变量的原子操作

3）会引发ABA问题

### 3、ABA问题

CAS算法实现了一个重要前提需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如一个线程one从内存位置V取出A，这时候**另一个线程tow也从内存中取出A，并且线程two进行了一些操作将值变成了B，然后线程two又将V位置的数据变成A**，这时候线程one进行CAS操作发现内存中仍然是A，然后线程one操作成功。

**尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。**



### 4、AtomicReference 原子引用

参见实现Lock

### 5、AtomicStampedReference原子版本号引用解决ABA问题

~~~java
/**
 * 解决ABA问题，原子版本号引用
 * @version 
 * @author 傅泉明
 */
public class ABADemo {
    // 初始化值0，版本0
    static AtomicStampedReference<Integer> reference = new AtomicStampedReference<Integer>(0, 0);
    
    public static void main(String[] args) throws Exception {
        CyclicBarrier barrier = new CyclicBarrier(2);
        Thread t1 = new Thread(() -> {
            // 将值0->1，版本0->1
            int stamp = reference.getStamp();
            try {
                barrier.await();// 多个线程一起等待读取到同一个版本，后再执行
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "," + reference.compareAndSet(0, 1, stamp, stamp + 1));
            // 将值1->0，版本1->2
            System.out.println(Thread.currentThread().getName() + "," + reference.compareAndSet(1, 0, reference.getStamp(), reference.getStamp() + 1));
        }, "t1");
        t1.start();
        
        Thread t2 = new Thread(() -> {
            int stamp = reference.getStamp();
            try {
                barrier.await();// 多个线程一起等待读取到同一个版本，后再执行
                // 等t1执行完成
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            // 将值0->1，版本0->1
            System.out.println(Thread.currentThread().getName() + "," + reference.compareAndSet(0, 1, stamp, stamp + 1));
        }, "t2");
        t2.start();
        t1.join();
        t2.join();
        System.out.println("reference=" + reference.getReference() + ",stamp=" + reference.getStamp());
    }
}
~~~







考察点：CAS->Usafe->CAS底层的思想->ABA->原子引用更新->如何规避ABA问题



## 五、Lock

实现Lock

~~~java
/**
 * 
 * @version 
 * @author 傅泉明
 */
public class MyLock implements Lock {
    // 1.锁的拥有者
    private AtomicReference<Thread> owner = new AtomicReference<Thread>(); 
    // 2.等待队列
    private LinkedBlockingQueue<Thread> waiter = new LinkedBlockingQueue<Thread>();
    
    @Override
    public void lock() {
        boolean flag = true;
        if (Thread.currentThread() == owner.get()) {// 已经有锁
            flag = false;
        }
        if (flag == true) flag = tryLock();
        if (flag == false) {
            waiter.offer(Thread.currentThread());// 进队列
            /* wait/notify 在synchronized使用
               condtion Lock中
               suspend/resume 弃用：容易死锁
               park/unpark
            */ 
            // 挂起当前线程
            System.out.println("park:" + Thread.currentThread().getName());
            LockSupport.park();
            lock();
        }
    }
    @Override
    public boolean tryLock() {
        return owner.compareAndSet(null, Thread.currentThread());
    }
    @Override
    public void unlock() {
        // 不是当前线程
        if (owner.get() != Thread.currentThread()) return;
        owner.compareAndSet(Thread.currentThread(), null);
        // 唤醒线程
        Thread waiterThread = waiter.poll();
        if (waiterThread != null) {
            LockSupport.unpark(waiterThread);             
            System.out.println("unpark:" + waiterThread.getName());
        }
    }
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {return false;}
    @Override
    public void lockInterruptibly() throws InterruptedException {}
    @Override
    public Condition newCondition() {return null;}
    
    public static void main(String[] args) {
        final TestLock test = new TestLock();
        for (int i = 0; i < 6; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    test.add();                  
                }
            }).start();
        }
    }

}
~~~

公平锁/非公平锁/可重入锁/递归锁/自旋锁谈谈你的理解？手写一个自旋锁

### 1、公平锁和非公平锁

并发包中ReentrantLock的创建可以指定构造函数的boolean类型来得到公平锁或非公平锁，默认是非公平锁。

公平锁：在并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则就加入到等待队列中，以后会按照FIFO的规则在从队列中取出自己。

非公平锁：上来就直接尝试占有锁，如果尝试失败，就在采用类似公平锁的方式。

**非公平锁的有点在于吞吐量比公平锁大。**

### 2、可重入锁（递归锁）

指同一线程在外层函数获得锁之后，内层递归函数仍然能获取该锁的代码（自动获取锁）

**线程可以进入如何一个它已经拥有锁的所有同步代码块。**

ReentrantLock/Synchronized都是可重入锁

**可重入锁最大的作用就是避免死锁**

### 3、自旋锁

指尝试获取锁的线程不会立即阻塞，而是采用循环的方式尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU

~~~java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while (!this.compareAndSeapInt(var1, var2, var5, var5 + var4));
    return var5;
}
~~~

### 4、独占锁和共享锁

独占锁：该锁一次只能被一个线程所所有。ReentrantLock和Synchronized都是独占锁。

共享锁：该锁可被多个线程持有。

ReentrantReadWriteLock是读锁共享锁，写锁是独占锁。读锁的共享锁可以保证并发读是高效，读写，写读，写写的过程是互斥的。

### 5、CountDownLatch

等待多个线程执行完成后，主线程才继续执行。

当一个或多个线程调用await方法时，调用线程会被阻塞。其他线程调用countDown方法会将计数器减1（调用countDown方法的线程不会阻塞），当计数器的值变为0时，因调用await方法被阻塞的线程才会被唤醒，继续执行。

### 6、CyclicBarrier

可循环使用的屏障，让一组线程到达一个屏障（也可以是同步点）时被阻塞，知道最后一个线程到达屏障时，屏障才会打开，所有被屏障阻塞的线程才会被唤醒，线程进入屏障通过CyclicBrarrier的await()方法。

### 7、Semaphore

信号量主要2个目的，一个用于多个共享资源的互斥使用，一个是并发线程数的控制。

~~~java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();// 抢占资源，自旋
                    Thread.sleep(1000);
                    System.out.println("占到资源" + Thread.currentThread().getName());
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();// 释放资源,给别人
                }
            }).start();
        }
    }
}
~~~

### 8、Synchronized和Lock区别

#### 1）原始构成-关键字

**synchronize是关键字属于JVM层面**

​	monitorenter（底层是通过moniter对象来完成，其实wait/notify等方法也依赖与monitor对象，只有在同步块或方法中才能调用wait/notify等方法）

​	monitorexit

**Lock是具体类java.util.concurrent.locks.lock是api侧面的锁**

#### 2）使用方法-释放锁

synchronize不需要用户手动释放锁，但synchronize代码执行完成后系统会自动释放锁。

ReentrantLock则需要用户手动释放锁，若无可能出现死锁。

#### 3）等待是否可中断

synchronize不可中断，除非抛出异常或者正常运行完成

ReentrantLock可中断，1.设置超时方法tryLock(long timeout, TimeUnit unit)

​                                         2.lockInterruptibly()放代码块中，调用interrupt()方法中断

#### 4）加锁是否公平

synchronize非公平锁

ReentrantLock两者都可以，默认非公平锁，

#### 5）唤醒线程组

synchronize没有

ReentrantLock用来实现分组唤醒需要唤醒的线程组。synchronize随机唤醒一个线程或唤醒全部线程。

### 9、Lock和Condition

~~~java
/**
 * 多线程按顺序调用，A->B->C三个线程启动
 * 1）A打印5次，B打印10次，C打印15次
 * 2）A打印5次，B打印10次，C打印15次
 * 3）轮询10次
 * @version 
 * @author 傅泉明
 */
public class LockAndConditionDemo {
    public static void main(String[] args) {
        LockData data = new LockData();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.println5();
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.println10();
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.println15();
        }, "C").start();
    }
}
class LockData {
    Lock lock = new ReentrantLock();
    volatile int count = 0;// A:0,B:1,C:2
    Condition c0 = lock.newCondition();
    Condition c1 = lock.newCondition();
    Condition c2 = lock.newCondition();
    public void println5() {
        lock.lock();
        try {
            while (count != 0) {// 不等于0，A等待
                c0.await();
            }
            count++;
            for (int i = 1; i <= 5; i++) System.out.println(Thread.currentThread().getName() + "," + i);
            c1.signalAll();// 唤醒B
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void println10() {
        lock.lock();
        try {
            while (count != 1) {// 不等于1，B等待
                c1.await();
            }
            count++;
            for (int i = 1; i <= 10; i++) System.out.println(Thread.currentThread().getName() + "," + i);
            c2.signalAll();// 唤醒C
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void println15() {
        lock.lock();
        try {
            while (count != 2) {// 不等于2，C等待
                c2.await();
            }
            count = 0;
            for (int i = 1; i <= 15; i++) System.out.println(Thread.currentThread().getName() + "," + i);
            c0.signalAll();// 唤醒A
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
~~~





## 六、AOP拦截

### 1、订单获取用户名称（订单、用户在不同的数据库）





## 七、性能测试工具

### 1、Apache JMeter

Apache JMeter是Apache组织开发的基于Java的压力测试工具。

用于对软件做压力测试，它最初设计用于Web应用测试，但后来扩展到其他测试领域

**JMeter可以用于对服务器、网络或对象模拟巨大的负载**，来自不同压力类别下测试它们的强度和分析整体性能。

#### 1）TPS

Transaction Per Second：每秒事务数，指服务器在单位时间内（秒）可以处理的事务数量

程序新增、提交表单-数据修改

#### 2）QPS

Query Per Second：每秒查询率，指服务器在单位时间内（秒）处理查询请求速率

API接口--1W/s

#### 3）PV

Page View：页面浏览量，通常是衡量一个页面甚至整个网站流量的重要指标

一个页面--n多个接口调用

#### 4）RT

Response Time/average Response Time：响应时间/平均响应时间，指一个事务花费多次时间完成

请求--服务端需要多久响应--这个问题最突出

### 2、问题排查操作步骤

#### 1）常规检查

**发下问题：前端埋点，预警 --- 多地用户 前端接口访问很慢**

CPU使用率、内存、网络、磁盘

一般都是Linux：TOP

不同地区：ping www.baidu.com

操作系统日志：tail -f /var/log/messages 文件



例子：Java突然挂掉，jvm进程突然没了？

**linux自我保护机制，OOM-killer（内存不足的情况，杀掉高内存的进程），这个操作会保留日志**

#### 2）java程序的问题

JDK工具--提供开发人员工具

java执行代码：

需要线程（线程创建不了了，死锁、）

​	排除（无法创建线程，会报错。cpu不执行线程，排除cpu繁忙）

jcmd：当前服务器运行了哪些jvm程序

jstack：查看jvm正在运行的线程信息

创建对象需要内存：内存不够就会GC垃圾回收--（STW）Stop The World--会挂起用户线程

jmap -heap JVM 进程ID

多线程并行异步Callable

**jps：查看当前java进程**

**jstack：查看jvm里线程**

### 3、死锁

~~~java
/**
 * 死锁实现
 * jps：查看当前java进程
 * jstack：查看进程里的线程
 * @version 
 * @author 傅泉明
 */
public class DeadlockDemo {
    public static void main(String[] args) {
        String lockA = "lockA";
        String lockB = "lockB";
        new Thread(new DeadlockData(lockA, lockB), "ThreadA").start();
        new Thread(new DeadlockData(lockB, lockA), "ThreadB").start();
    }
}
class DeadlockData implements Runnable {
    private String lockA;
    private String lockB;
    public DeadlockData(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }
    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() + ",获取" + lockA + ",正在获取" + lockB);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() + ",获取" + lockB + ",正在获取" + lockA);
            }
        }
    }
}
~~~





## 八、volatile

### 1、volatile是Java虚拟机提供的轻量级的同步机制

1）保证可见性

**2）不保证原子性**

3）禁止指令重拍

### 2、JMM

JMM（Java内存模型Java Memory Model，简称JMM）本省是一种抽象的概念并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组的元素）的访问方式。

JMM同步的规定：

1）线程解锁前，必须把共享变量的值刷新回主内存

2）线程加锁前，必须读取主内存的最新值到自己的工作内存

3）加锁解锁是同一把锁

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定私有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，**但线程对变量的操作（读取赋值等）必须在工作内存中进行，首先将变量从主内存拷贝到自己的工作内存空间，然后对变量进行草，操作完成后再将变量写回主内存**，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存的**变量副本拷贝，因此不同线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成。**

#### 1）可见性

**指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值**

~~~java
/**
 * volatile
 * 1、可见性：多线程同步变量
 * 2、原子性：
 * @version 
 * @author 傅泉明
 */
public class VolatileDemo {
    public void volatileTest() throws Exception {
        // 可见性
        final Data data = new Data();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                data.setSize();
                System.out.println("Thread:" + Thread.currentThread().getName() + ",变更data.size=" + data.size);
            }
        }).start();
        System.out.println(data.size);
        while (data.size == 0) {
            // TODO 说主线程会一直卡主，获取不到线程更新后的值，可是本地测试后主线程能获取到最新值
            Thread.sleep(500);
        }
        System.out.println(data.size);
    }
    public static void main(String[] args) throws Exception {
        // 可见性
        VolatileDemo demo = new VolatileDemo();
        demo.volatileTest();        
    }
}
class Data {
    int size = 0;// 无 volatile 修饰
    public void setSize() {
        this.size = 10;
    }
    public void addSize() {
        this.size++;
    }
}
~~~

#### 2）原子性

**即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。**

size++被拆分成3个指令：

执行GETFIELD拿到主内存中的原始值size

执行IADD进行加1操作

执行PUTFIELD把工作内存中的值写回主内存中

当多个线程并发执行PUTFIELD指令的时候，会出现写回主内存覆盖问题，所以才会导致最终结果不为10000，volatile不能保证原子性。

size++在多线程下是非线程安全，如何不加synchronize解决？

1）synchronize

2）Lock

3）AtomicInteger

~~~java
/**
 * volatile
 * 1、可见性：多线程同步变量
 * 2、原子性：
 * @version 
 * @author 傅泉明
 */
public class VolatileDemo {    
    public static void main(String[] args) throws Exception {
        // 原子性
        int size = 10;
        CountDownLatch latch = new CountDownLatch(size);
        Data data = new Data();
        for (int i = 0; i < size; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    data.addSize();
                }
                latch.countDown();
            }).start();
        }
        // 查看最终结果值
        latch.await();
        System.out.println("data.size=" + data.size);
    }
}
class Data {
    int size = 0;// 无 volatile 修饰
    public void setSize() {
        this.size = 10;
    }
    public void addSize() {
        this.size++;
    }
}
~~~

#### 3）有序性

**即程序执行的顺序按照代码的先后顺序执行**

##### 1、指令重排有序性：

计算机在执行程序时，为了提高性能，编译器和处理器常常会做指令重排，一般分为以下三种：

**源代码->编译器优化重排->指令并重排->内存系统重排->最终执行的指令**

单线程环境里面确保程序最终执行结果和代码顺序执行结果一致。

处理器在进行指令重排序时必须考虑指令之间的**数据依赖性**

多线程环境中线程交替执行，由于编译器指令重排的存在，两个线程使用的变量能否保证一致性是无法确认的，结果无法预测。



指令重排案例分析one：

```java
public void mySort() {
    int x = 11; // 语句1
    int y = 12; // 语句2
    x = x + 5; // 语句3
    y = x * x; // 语句4
}
// 指令重排之后，代码执行顺序有可能是以下几种可能？
// 语句1 -> 语句2 -> 语句3 -> 语句4
// 语句1 -> 语句3 -> 语句2 -> 语句4
// 语句2 -> 语句1 -> 语句3 -> 语句4
// 问题：请问语句4可以重排后变为第1条吗？
// 不能，因为处理器在指令重排时必须考虑指令之间数据依赖性。
```
---------------------
指令重排案例分析three：

```java
public class BanCommandReSortSeq {
	int a = 0; 	
	boolean flag = false; 
	public void methodOne() {
    	a = 1;  // 语句1
    	flag = true;    // 语句2
    	// methodOne发生指令重排，程序执行顺序可能如下：
    	// flag = true;    // 语句2
    	// a = 1;  // 语句1
    }
    public void methodTwo() {
    	if (flag) {
        	a = a + 5;  // 语句3
    	}
    	System.out.println("methodTwo ret a = " + a);
    }
}
// 多线程环境中线程交替执行，由于编译器指令重排的存在，两个线程使用的变量能否保证一致性是无法确认的，结果无法预测。
// 多线程交替调用会出现如下场景：
// 线程1调用methodOne，如果此时编译器进行指令重排
// methodOne代码执行顺序变为：语句2（flag=true） -> 语句1（a=5）
// 线程2调用methodTwo，由于flag=true，如果此时语句1还没有执行（语句2 -> 语句3 -> 语句1 ），那么执行语句3的时候a的初始值=0
// 所以最终a的返回结果可能为 a = 0 + 5 = 5，而不是我们认为的a = 1 + 5 = 6;
```

##### 2、指令重排原理

volatile实现禁止指令重排优化，从而避免多线程环境下程序出现乱序执行的现象。

内存屏障（Memory Barrier）又称内存栅栏，是一个CPU指令，它的作用有两个：

- 保证特定操作执行的顺序性；
- 保证某些变量的内存可见性（利用该特性实现volatile内存可见性）

由于编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier则会告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重排，也就是说**通过插入内存屏障，就能禁止在内存屏障前后的指令执行重排优化。内存屏障另外一个作用就是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本**。

左边：写操作场景：先LoadStore指令，后LoadLoad指令。

右边：读操作场景：先LoadLoad指令，后LoadStore指令。



### 3、哪里用到过volatile？

单例模式(DCL-Double Check Lock双端检锁机制)

~~~java
/**
 * 单例：DCL-Double Check Lock双端检锁机制
 * @version 
 * @author 傅泉明
 */
public class SingleDemo {
    private static volatile SingleDemo instance;
    private SingleDemo() {
        System.out.println("实例化SingleDemo");
    }
    public static SingleDemo getInstance() {
        if (instance == null) {
            synchronized (SingleDemo.class) {
                if (instance == null) {
                    instance = new SingleDemo();
                }
            }
        }
        return instance;
    }
    public static void main(String[] args) throws Exception {
        int count = 1000;
        CountDownLatch latch = new CountDownLatch(count);
        for (int i = 0; i < count; i++) {
            new Thread(() -> {
                SingleDemo.getInstance();
                latch.countDown();
            }).start();
        }
        latch.await();
        System.out.println("end");
    }
}
~~~

需要volatile关键字的原因是，在并发情况下，如果没有volatile关键字，在第5行会出现问题。instance = new SingleDemo();可以分解为3行伪代码

原因：在某个线程执行到第一次检测，读取到的instance不为null时，instance对象可能没有完成初始化。

instance = new SingletonDemo();可以分为以下3步完成（伪代码）

memory = allocate();    // 1.分配内存对象空间

instance = (memory);   // 2.初始化对象

instance = memory;     // 3.设置instance指向刚分配的内存地址，此时instance != null

由于步骤2和步骤3不存在数据依赖关系，而且无论重排前还是重排后程序的执行结果在单线程中并没有改变，因此这种重排优化是允许的。

memory = allocate();    // 1.分配内存对象空间

instance = memory;     // 3.设置instance指向刚分配的内存地址，此时instance != null，但是对象初始化没有完成，就相当于这个地址空间内没有实际的值。

instance = (memory);   // 2.初始化对象

## 九、集合不安全问题

~~~java
/**
 * 集合不安全问题,list，set，map
 * @version 
 * @author 傅泉明
 */
public class CollectionNoSafeDemo {

    public static void main(String[] args) throws Exception {
        List<String> list = new ArrayList<>();// 线程不安全
//        List<String> list = new Vector<>();
//        List<String> list = Collections.synchronizedList(new ArrayList<>());
//        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }).start();
        }
        // CopyOnWriteArraySet底层也是CopyOnWriteArrayList
        Set<String> set = new CopyOnWriteArraySet<>();
        // HashSet底层也是HashMap，value都是同一对象
        HashSet<String> hashSet = new HashSet<>();
        //
        Map<String, String> map = new ConcurrentHashMap<>();
    }
    // java.util.ConcurrentModificationException
    /**
     * ArrayList 线程不安全
     * 1.故障现象
     *      java.util.ConcurrentModificationException
     * 2.导致原因
     *      一个线程正在修改，另一个线程抢夺，导致并发修改异常
     * 3.解决方案
     *      new Vector<>();
     *      Collections.synchronizedList(new ArrayList<>());
     *      new CopyOnWriteArrayList<>();// 写时复制
     * 4.优化建议
     */
    /**
     * CopyOnWrite容器即写时复制的容器，将当前容器Object[]进行Copy，复制出新的容器Object[] newElements，然后在新容器里添加元素
     * 之后将原容器的引用指向新容器。写的时候加锁，读的时候不需要。
     * public boolean add(E e) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len + 1);
                newElements[len] = e;
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();
            }
        }
     */
}
~~~

## 十、受检异常和非受检异常的区别

继承Execption是受检异常，要捕获异常或向上抛出异常，如IOExecption

继承RuntimeExecption是非受检异常，不要捕获或向上抛出异常

在接口改造添加异常时，为不影响调用需要捕获新异常，将新异常添加为RuntimeExecption。

## 十一、JVM

### 1、JVM内存结构

#### 1）JVM体系概述





#### 2）Java8以后的JVM





### 2、GC的作用域



### 3、常见垃圾回收算法












































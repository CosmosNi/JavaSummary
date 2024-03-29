---
typora-copy-images-to: ..\image
---

# jdk内置JUC组件

###### 1.1 JUC核心组件AQS

AQS是类AbstractQueuedSynchronizer的简称（以下均以AQS代替），提供了一种实现堵塞锁（独占锁）和一系列FIFO等待队列（共享锁）的同步框架。
独占锁：此代码有且只有一个线程能够执行，如ReentrantLock
共享锁：多个线程可同时获取锁，如Semaphore/CountDownLatch



###### 1.2 Lock相关概念

Lock作用:

- 锁用来保护代码片段，任何时刻只能有一个线程执行被保护的代码。
- 锁可以管理试图进入被保护代码段的线程。
- 锁可以拥有一个或多个相关的条件对象。
- 每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程。

公平与非公平锁: 公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。而非公平锁是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样的就是先来的不一定先得到锁，这个方式肯呢过造成某些线程一直拿不到锁，结果也就是不公平的了。

###### 1.2.1 ReentrantLock (默认非公平锁)

 重入锁（ReentrantLock）是一种递归无阻塞的同步机制。 它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放。这模仿了 synchronized 的语义；如果线程进入由线程已经拥有的监控器保护的 synchronized 块，就允许线程继续进行，当线程退出第二个（或者后续） synchronized 块的时候，不释放锁，只有线程退出它进入的监控器保护的第一个 synchronized 块时，才释放锁。

常用方法：

- lock()：
  - 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。
  - 如果当前线程已经保持该锁，则将保持计数加 1，并且该方法立即返回。
  - 如果该锁被另一个线程保持，则出于线程调度的目的，禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态，此时锁保持计数被设置为 1
- lockInterruptibly()
  - 如果当前线程未被中断，则获取锁。
  - 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。
  - 如果当前线程已经保持此锁，则将保持计数加 1，并且该方法立即返回。
  - 如果锁被另一个线程保持，则出于线程调度目的，禁用当前线程，并且在发生以下两种情况之一以前，该线程将一直处于休眠状态(锁由当前线程获得；或者 其他某个线程中断当前线程。 )
  - 如果当前线程获得该锁，则将锁保持计数设置为 1。
  - 此方法是一个显式中断点，所以要优先考虑响应中断，而不是响应锁的普通获取或重入获取。
- tryLock():仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
  - 如果该锁没有被另一个线程保持，并且立即返回 true 值，则将锁的保持计数设置为 1。即使已将此锁设置为使用公平排序策略，但是调用 tryLock() 仍将 立即获取锁（如果有可用的），而不管其他线程当前是否正在等待该锁。在某些情况下，此“闯入”行为可能很有用，即使它会打破公平性也如此。如果希望遵守此锁的公平设置，则使用 tryLock(0, TimeUnit.SECONDS) ，它几乎是等效的（也检测中断）。
    - 如果当前线程已经保持此锁，则将保持计数加 1，该方法将返回 true。
    - 如果锁被另一个线程保持，则此方法将立即返回 false 值。

###### 1.2.2 Conditon

Condition主要是用来处理线程之间的通信，当满足某一条件时，唤醒其他的线程。
主要配合重入锁完成等待唤醒的操作。



###### 1.3 synchroized

相关定义：
- 调用关键字synchroized生命的方法一定是排队运行的

- 避免数据出现交叉的情况，使用synchroized关键词进行同步
- 关键词synchroized拥有锁重入的功能 ，也就是在使用synchroized时，当一个线程得到一个对象锁后，在此请求此对象锁时是可以再次得到该对象锁的。
- 当一个线程执行的代码出现异常时，其所持有的锁会自动释放。
- 同步不可以被继承。
- synchroized同步块 锁非this对象相比synchroized（this）更加灵活，当一个方法中有多个同步块时，不用竞争this对象锁。

关于synchroized（非this对象x）的写法是将x对象本身作为"对象监视器"，有如下三个结论：

- 当多个线程同时执行synchroized（X）{}同步代码块时呈同步效果。
- 当其他线程执行x对象中synchroized同步方法时呈同步效果。
- 当其他线程执行x对象方法中的synchroized（this）代码块时也呈现同步效果。
- 如果其他线程调用不加synchroized关键字的方法时，还是异步调用。
- synchroized关键字加到非static静态方法是给对象上锁，而教导static方法上则是对Class类加锁。
- 对于String对象，不要用作对象锁。String a="A",String b="A"，a==b，导致线程会使用同一个对象锁
- 在设计程序时，要避免双方互相持有对方锁的情况，会导致死锁。

###### 1.4 volitile

volitile只保证可见性，不保证原子性。

使用场景：

1. 写入变量时并不依赖变量的当前值。或者能够确定保证只有单一的线程能修改变量的值。
2. 变量不需要与其他状态变量共同参与不变约束。 
3. 访问变量的时候，没有其他原因需要加锁。



volatile和synchronized比较

1. 关键字volatile是线程同步的轻量实现，所以volatile性能肯定比synchronized好，并且volatile只能修饰于变量，而synchronized可以修饰方法以及代码块。
2. 多线程访问volatile不会发生堵塞，而synchronized会出现堵塞。
3. volatile只能保证数据的可见性，但不能保证原子性；而synchronized可以保证原子性，也可间接保证可见性，因为它会将私有内存和公共内存中的数据做同步
4. volatile解决的是变量在多个线程之间的可见性，而synchronized解决的是多个线程之间访问资源的同步性。



###### 1.5 ThreadLocal（InheritableThreadLocal 提供父子线程变量共享）

Threadlocal提供了get和set的访问器，为每个使用它的线程维护一份单独的拷贝。所以get返回的都是当前线程设置的最新值。

ThreadLocal在内部维护了一个ThreadMap用来映射线程的独有变量。

- 一个Thread有且仅有一个ThreadLocalMap对象
- 一个Entry对象的key弱引用指向一个ThreadLocal对象。
- 一个ThreadLocalMap对象可以被多个线程所共享。
- ThreadLocal对象不持有Value，Value由线程的Entry对象持有。



ThreadLocal的弊端：

- 脏数据：线程复用会产生脏数据。由于线程池会重用Thread对象，那么和Thead绑定的静态属性ThreadLocal变量也会被重用。
- 内存泄露：在源码注释中提示使用static关键字来修饰ThreadLocal。在此场景下，寄希望于TheadLocal对象失去引用后，触发弱引用机制来回收Entry的Value就不现实了。

解决方案即在每次用完ThreadLocal时，必须及时调用remove（）方法进行清理



![1568192027802](../image/1568192027802.png)

###### 1.6 Future+callable

所谓异步调用其实就是实现一个可无需等待被调用函数的返回值而让操作继续运行的方法。在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果。

CompletableFuture:

- supplyAsync：执行一个异步请求，并返回一个future
- thenApply：对结果应用一个函数
- thenCompose：对结果调用函数并执行返回的future
- handle：处理结果或错误
- thenAccept： 类似于 thenApply, 不过结果为 void
- whenComplete：类似于 handle, 不过结果为 void
- thenRun：执行 Runnable, 结果为 void
- thenCombine:执行两个动作并用给定函数组合结果
- thenAcceptBoth:与 thenCombine 类似， 不过结果为 void
- runAfterBoth:两个都完成后执行_able
- applyToEither:得到其中一个的结果时， 传入给定的函数
- acceptEither:与 applyToEither 类似， 不过结果为 void
- runAfterEither:其中一个完成后执行 runnable
- static allOf:所有给定的 future 都完成后完成，结果为 void
- static anyOf:任意给定的 future 完成后则完成，结果为 void

###### 1.7 SynchronousQueue（同步队列）

同步队列是一种将生产者与消费者线程配对的机制。当一个线程调用 SynchronousQueue 的 put 方法时，它会阻塞直到另一个线程调用 take 方法为止， 反之亦然。 与 Exchanger 的情 况不同， 数据仅仅沿一个方向传递，从生产者到消费者。 即使 SynchronousQueue 类实现了 BlockingQueue 接口， 概念上讲， 它依然不是一个队 列。它没有包含任何元素，它的 size 方法总是返回 0。



###### 1.8 堵塞队列

用处： 对于许多线程问题， 可以通过使用一个或多个队列以优雅且安全的方式将其形式化。生产者线程向队列插人元素， 消费者线程则取出它们。使用队列，可以安全地从一个线程向另一个线程传递数据。

常用堵塞队列

- ArrayBlockingQueue：构造一个带有指定容量和公平性设置的堵塞队列。该队列用循环数组实现。
- LinkedBlockingQueue/LinkedBlockingDeque：构造一个无上限的堵塞队列或双向队列，用链表实现。
- DelayQueue：构造一个包含 Delayed 元素的无界的阻塞时间有限的阻塞队列。只有那些延迟已经超过时间的元素可以从队列中移出。
- PriorityBlockingQueue：构造一个无边界阻塞优先队列，用堆实现。
- ConcurrentLinkedQueue：构造一个可以被多线程安全访问的无边界非阻塞的队列。
- ConcurrentSkipListSet：构造一个可以被多线程安全访问的有序集。第一个构造器要求元素实现 Comparable 接口。



###### 1.9 CountDownLatch闭锁

- 确保一个计算不被执行，直到它需要的资源初始化。
- 确保一个服务不会开始，直到它依赖的其他服务都已经开始。
- 等待，直到活动的所有部分都为继续处理做好充分准备。 CountDownLatch是一个灵活的闭锁实现。允许一个或多个线程等待一个事件集的发生。闭锁状态包括一个计数器，初始化为一个整数。 用来表现需要等待的事件数。countDown方法针对计数器做减操作，表示一个事件已经发生了，而await方法等待计数器达到零，此时所有需要等待的事件都已经发生。如果计数器入口时值为非零，await会一直堵塞直到计数器为零，或者等待线程中断以及超时。



###### 1.10 CyclicBarrier同步屏障

类似于闭锁。与闭锁不同之处在于，所有的线程必须同时到达关卡点，才能继续处理。 闭锁等待的是事件；而同步屏障等待的是其他的线程。 常用示例比如：可将一个任务分割成多个子部分，然后再整合。



###### 1.11 Semaphore计数信号量

- 用来控制能够同时访问某特定资源的活动的数量。
- 计数信号量可以用来实现资源池或者给一个容器限定边界。
- 一个Semaphore管理一个有效的许可，许可的除湿量通过构造函数传递给semaphore活动能够获得许可（只要还有剩余许可），并在使用之后释放许可，如果没有可用的许可，则acquire会被堵塞，直到有可用的为止。
- 常见的信号量使用即数据库连接池。



###### 1.12 Exchanger交换器

当两个线程在同一个数据缓冲区的两个实例上工作的时候， 就可以使用交换器 ( Exchanger) 典型的情况是， 一个线程向缓冲区填人数据， 另一个线程消耗这些数据。当它们都完成以后，相互交换缓冲区。



###### 1.13 线程池相关参数

核心ThreadPoolExecutor参数：

- corePoolSize：指定线程池中的线程数量
- maximumPoolSize：指定了线程池中的最大线程数量。
- keepAliveTime：当线程池线程数量超过corePoolSize，多余的空闲线程的存活时间，即超过corePoolSizde的空闲线程，在多长时间内会被销毁。
- unit：keepAliveTime的单位
- workQueue：任务队列，被提交但未被执行的任务。当请求的线程数大于maximumPoolSize时，线程进入BlockingQueue阻塞队列。
- threadFactory:线程工厂，用于创建线程，一般用默认的既可
- handler：拒绝策略。当任务太多来不及处理，如何拒绝任务。

workQueue说明： 是一个BlockingQueue接口对象。仅用于存放Runnable对象。

- 直接提交队列：该功能由SynchronousQueue对象提供。SynchronousQueue没有容量，每一个插入操作都要等待一个相应的删除操作，同理删除。如果使用SynchronizeQueue，则提交的任务不会被真实的保存，而总是将新任务提交给线程执行，如果没有空闲的线程，则尝试创建新的线程，如果线程已经达到最大值，则执行拒绝策略。
- 有界的任务队列：ArrayBlockQueue实现。当有新的任务需要执行，如果线程池的实际线程数小于corePoolSize，则会优先创建新的线程，若大于corePoolSize，则会将新任务加入等待队列，若等待队列已满，无法加入，则在总线程不大于maximumPoolSize的前提下，创建新的线程执行任务。若大于maximumPoolSize，则执行拒绝策略。
- 无界的任务队列：LinkBlockQueue类实现。与有界队列相比，除非系统资源耗尽，否则无界队列不存在任务入队失败的情况，当有新的任务到来，系统的线程数小于corePoolSize，线程池会生成新的线程执行任务，当系统的线程数达到corePoolSize，就不会继续增加了。若后续仍有新的任务加入，而有没有空闲的线程资源，则任务直接进入队列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。
- 优先任务队列：PriorityBlockQueue实现。可以控制任务执行的先后顺序。它是一个特殊的无界队列。

拒绝策略说明：（可扩展RejectedExecutionHandler接口）

- AbprtPolicy：该策略会直接抛出异常，阻止系统工作
- CallRunsPolicy：只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能急剧下降。
- DiscardOldestPolicy：该策略将丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。
- DiscardPolicy：该策略默默丢弃无法处理的任务，不予任何处理。

扩展：ThreadPoolExecutor提供了beforeExecute(),afterExecute和terminated()三个方法用来对线程池进行控制。



线程池使用注意点：

- 合理设置各类参数，应根据实际业务场景来设置合理的工作线程数
- 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程
- 创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。



ThreadPoolExecutor源码：

```java
	//Integer共有32位，最右边29位表示工作线程数，最左边三位标识线程池状态。即，3个二进制可以表示从0至7的8个不同数值
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    //此状态表示线程池能接受新任务
    private static final int RUNNING    = -1 << COUNT_BITS;
	//此状态不再接受新任务，但可以继续执行队列中的任务
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
	//此状态全面拒绝，并中断正在处理的任务
    private static final int STOP       =  1 << COUNT_BITS;
	//此状态表示所有任务已经被终止
    private static final int TIDYING    =  2 << COUNT_BITS;
	//此状态表示已清理完现场
    private static final int TERMINATED =  3 << COUNT_BITS;

    // Packing and unpacking ctl
	//表示线程池当前处于stop状态
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
	//工作线程数
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }
```

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
    	//返回包含线程数以及线程池状态的Integer类型数值
        int c = ctl.get();
    	//如果工作线程数小于核心线程数，则创建线程任务并执行
        if (workerCountOf(c) < corePoolSize) {
            //重要方法
            if (addWorker(command, true))
                return;
            //如果创建失败，防止外部已经在线程池中加入新任务，重新获取下
            c = ctl.get();
        }
    	//只有线程池处于RUNNING状态，才执行后半句：置入队列
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            //如果线程池不是Running状态，则将刚加入队列的任务移除
            if (! isRunning(recheck) && remove(command))
                reject(command);
            //如果之前的线程已被消费完，新建一个线程
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
    	//核心池和队列都已满，尝试创建一个新线程
        else if (!addWorker(command, false))
            //如果addWorker返回的是false，即创建失败，则唤醒拒绝策略
            reject(command);
    }
```

```java
/**
 * 根据当前线程池状态，检查是否可以添加新的任务线程，如果可以则创建并
 * 启动任务，如果一切正常则返回true，返回false的可能性如下：
 * 1. 线程池没有处于RUNNING状态
 * 2. 线程工厂创建新的任务线程失败。
 *
 * firstTask：外部启动线程池时需要构造的第一个线程，它是线程的母体
 * core：新增工作线程时的判断指标，解释如下
 *      true：表示新增工作线程时，需要判断当前RUNNING状态的线程是否少于corePoolSize
 *      false：表示新增工作线程时，需要判断当前RUNNING状态的线程是否少于maximumPoolSize
 */
private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            //如果RUNNING状态，则条件为假，不执行后面的判断
            //如果时STOP及之上的状态，或者firstTask初始线程不为空，或者队列为空
            //都会直接返回创建失败
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
                //如果超过最大允许线程数则不能再添加新的线程
                //最大线程数不能超过2^29，否则会影响左边3位的线程池状态值
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                //当前活动线程数+1
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                //线程池状态和工作线程数是可变的，需要经常提取这个最新值
                c = ctl.get();  // Re-read ctl
                //如果已经关闭，则再次从rerty标签处进入
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }
		//开始创建工作线程
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            //利用Worker构造方法中的线程池工厂创建线程，并封装成工作线程Worker对象
            w = new Worker(firstTask);
            //注意这是Worker中的属性对象thread
            final Thread t = w.thread;
            if (t != null) {
                //在进行ThreadPoolExecutor的敏感操作时
                //都需要持有主锁，避免在添加和启动线程时被干扰
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        //整个线程池在运行期间的最大并发任务个数
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    //注意，并非线程池的execute的command参数指向的线程
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                //线程启动失败，将工作线程计数再减回去
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```


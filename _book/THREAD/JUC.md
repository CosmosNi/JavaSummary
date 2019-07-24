# jdk内置JUC组件

###### 1.1 Lock+Condition

Lock作用:

- 锁用来保护代码片段，任何时刻只能有一个线程执行被保护的代码。
- 锁可以管理试图进入被保护代码段的线程。
- 锁可以拥有一个或多个相关的条件对象。
- 每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程。

公平与非公平锁: 公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。而非公平锁是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样的就是先来的不一定先得到锁，这个方式肯呢过造成某些线程一直拿不到锁，结果也就是不公平的了。

ReentrantLock (默认非公平锁): 重入锁（ReentrantLock）是一种递归无阻塞的同步机制。 它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放。这模仿了 synchronized 的语义；如果线程进入由线程已经拥有的监控器保护的 synchronized 块，就允许线程继续进行，当线程退出第二个（或者后续） synchronized 块的时候，不释放锁，只有线程退出它进入的监控器保护的第一个 synchronized 块时，才释放锁。

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



###### 1.2 synchroized

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
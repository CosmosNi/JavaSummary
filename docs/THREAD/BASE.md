# 并发编程相关知识点

###### 1.1 线程的状态

1. New（新创建）：新创建一个新的线程对象对象后，在调用他的start（）方法，系统会为此线程分配CPU资源，使其处于Runnable（可运行状态），这是一个准备运行的阶段。如果线程抢占到CPU资源，此线程就处于Running（运行）状态。
2. Runnable（可运行） Runnable状态和Running状态可相互切换，因为有可能线程运行一段时间后，有其他高优先级的线程抢占了CPU资源，这时Running状态变成了Runnable状态。 线程进入Runnable状态大体分为如下几种情况：
   - 调用sleep（）方法后经过的时间超过了指定的休眠时间
   - 线程调用的堵塞IO已经返回，堵塞方法执行完毕
   - 线程成功地获取了试图同步的监视器
   - 线程处于等待某个通知，其他线程发出了通知。
   - 处于挂起状态的线程调用了resume回复方法。
3. Blocked（被堵塞） 例如遇到一个IO操作，此时CPU处于空闲状态，可能会转而把CPU时间片分配给其他的线程，这是也可以成为“暂停”状态。
   - 线程调用sleep（）方法，主动放弃占用的处理器资源。
   - 线程调用了堵塞式IO方法，在该方法返回前，该线程堵塞
   - 线程试图获得一个同步监视器，但该同步监视器正被其他线程所持有。
   - 线程等待某个通知。
   - 程序调用了suspend方法将该线程挂起。此方法容易导致死锁，尽量避免使用该方法。
4. run（）：运行结束进入销毁阶段，整个线程执行完毕。
5. join（）：使所属线程对象x正常执行run（）方法中的任务，而使当前线程z无限期堵塞，等待线程x销毁后再继续执行线程z后面的代码。join具有使线程排队运行的作用，有些类似同步的运行效果。join与synchronized的区别是：join在内部使用wait方法进行等待，而synchronized使用的“对象监视器”原理做同步。
6. Waiting（等待）
7. Timed（waiting）
8. Terminated（被终止）
# JUC核心之AQS

###  Node节点（线程状态维护）
AQS中维护线程状态是通过一个内部类Node来维护的。源码如下

```java
static final class Node {
        //节点在共享模式下的标记
        static final Node SHARED = new Node();
        //节点在独占锁模式下的标记
        static final Node EXCLUSIVE = null;
        //标志着线程被取消
        static final int CANCELLED =  1;
        //标志着后继线程(即队列链表的下一个节点)需要被阻塞.
        static final int SIGNAL    = -1;
        //标志着线程在Condition条件上等待阻塞
        static final int CONDITION = -2;
        //标志着下一个acquireShared方法线程应该被允许，即锁可向下一级传播。(用于共享锁)
        static final int PROPAGATE = -3;
        //维护锁的状态
        volatile int waitStatus;

        //前驱节点
        volatile Node prev;
        //后继结点      
        volatile Node next;
        //获取同步状态的线程,进入此节点队列的线程
        volatile Thread thread;
        //等待节点的后继节点。如果当前节点是共享的，那么这个字段是一个SHARED常量，也就是说节点类型（独占和共享）和等待队列中的后继节点共用一个字段。
        Node nextWaiter;

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        //返回前驱节点
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {   
        }
        //Used by addWaiter
        Node(Thread thread, Node mode) {
            this.nextWaiter = mode;
            this.thread = thread;
        }
        // Used by Condition
        Node(Thread thread, int waitStatus) {
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```






###  独占锁
四维导图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711140810617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjYxNzk4,size_16,color_FFFFFF,t_70)
1. 获取锁

```java
public final void acquire(int arg) {
     //尝试获取锁资源，失败则创建等待队列，此处的tryAcquire由子类进行实现
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        //中断当前线程
        selfInterrupt();
}
```

2. 将Node加入等待队列

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    //记录当前尾节点
    Node pred = tail;
    //此处为了提高性能，直接进行入队操作，失败则继续处理（多线程下竞争）
    if (pred != null) {
       //设置前驱节点
        node.prev = pred;
       // CAS操作，多个线程同时进行入队，只有一根线程能入队成功，其余的继续执行
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //上一步失败的节点入队
    enq(node);
    return node;
}
```



3. 节点加入队尾

```java
private Node enq(final Node node) {
//自旋锁，失败重试
    for (;;) {
    	//记录队尾元素
        Node t = tail;
        //初始化队列
        if (t == null) {
          //CAS操作，只有一个线程可以初始化头结点成功，其余的都要重复执行循环体
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
           //入队的节点指向队尾，此处会有多个节点指向队尾
            node.prev = t;
            //cas操作，保证多线程下只有一个节点真正指向队尾节点，其余失败重试
            if (compareAndSetTail(t, node)) {
                t.next = node;
                //该循环体唯一退出的操作，就是入队成功（否则就要无限重试）
                return t;
            }
        }
    }
}
```



4. 将节点加入等待队列后，先判断是否能够获取锁的资源，不能则进入挂起状态等待唤醒

```java
final boolean acquireQueued(final Node node, int arg) {
        //锁资源获取失败标记位
        boolean failed = true;
        try {
            //线程被中断标识
            boolean interrupted = false;
            // 自旋锁等待被唤醒
            for (;;) {
                //获取当前节点的前置节点
                final Node p = node.predecessor();
                //如果前置节点就是头结点，则尝试获取锁资源
                if (p == head && tryAcquire(arg)) {
                    //将当前节点设置为头结点
                    setHead(node);
                    p.next = null; //帮助GC
                    //表示锁资源成功获取
                    failed = false;
                    //返回中断标记，表示当前节点是被正常唤醒还是被中断唤醒
                    return interrupted;
                }
                //如果没有获取锁成功，则进入挂起逻辑
                //无论被中断或者正常唤醒，都会进行重新获取锁。成功则释放，失败则挂起
                // acquireInterruptibly不同在于，线程堵塞时被中断会抛出异常
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            //获取锁失败处理逻辑
            if (failed)
                cancelAcquire(node);
        }
    }
```

5. 检测节点状态，判断是否可进入挂起状态，同时剔除前边已经取消的节点
```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取前置节点的waitStatus
        int ws = pred.waitStatus;
        //如果处于等待唤醒状态，则返回为true进行堵塞
        if (ws == Node.SIGNAL)
            return true;
        if (ws > 0) {
          //循环查找没有被取消的前节点，相当于把之前取消的节点从队列中剔除出去
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
       		//将前置节点状态设置Node.SIGNAL，执行上一步的自旋，进行堵塞
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```
6. 堵塞线程
```
    private final boolean parkAndCheckInterrupt() {
      //堵塞线程
    LockSupport.park(this);
    //被唤醒之后，返回中断标记，即如果是正常唤醒则返回false，如果是由于中断醒来，就返回true
    return Thread.interrupted();
}
```
7. 获取锁或则加入等待队列失败后的处理

```java
private void cancelAcquire(Node node) {
        if (node == null)
            return;

        node.thread = null;

        Node pred = node.prev;
        // 跳过所有已经取消的前置节点，跟上面的那段跳转逻辑类似
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        //取得的未取消节点的后一节点
        Node predNext = pred.next;
        //把当前节点waitStatus置为取消，这样别的节点在处理时就会跳过该节点
        node.waitStatus = Node.CANCELLED;

        //如果当前是尾节点，则直接删除，即出队
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                //如果当前节点的前置节点不是头节点且它后面的节点等待它唤醒（waitStatus小于0），如果当前节点的后继节点没有被取消就把前置节点跟后置节点进行连接
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
```


8. 锁的释放

```java
public final boolean release(int arg) {
  // 调用tryRelease方法，尝试去释放锁，由子类具体实现
    if (tryRelease(arg)) {
        Node h = head;
        //如果队列头节点的状态不是0，那么队列中就可能存在需要唤醒的等待节点。
        //
        if (h != null && h.waitStatus != 0)
            // 唤醒线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

9. 线程唤醒

```java
private void unparkSuccessor(Node node) {

        int ws = node.waitStatus;
        //如果当前节点的状态小于0，则将该节点置为初始状态，标识节点已完成
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
              // 如果下一个节点为null，或者状态是已取消，那么就要寻找下一个非取消状态的节点
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 从队列尾向前遍历，直到遍历到node节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
           //唤醒线程
            LockSupport.unpark(s.thread);
    }
```

 ### 共享锁的实现
 1.释放共享锁
```java
private void doAcquireShared(int arg) {
      // 创建一个共享锁节点
       final Node node = addWaiter(Node.SHARED);
       boolean failed = true;
       try {
           boolean interrupted = false;
           for (;;) {
              // 获取下一节点
               final Node p = node.predecessor();
               //前节点为头结点
               if (p == head) {
                 //尝试释放共享锁
                   int r = tryAcquireShared(arg);
                   if (r >= 0) {
                        //获取锁后的唤醒操作
                       setHeadAndPropagate(node, r);
                       p.next = null; // help GC
                       //如果是被中断唤醒，则设置中断标记为
                       if (interrupted)
                           selfInterrupt();
                       failed = false;
                       return;
                   }
               }
               //挂起逻辑同独占锁
               if (shouldParkAfterFailedAcquire(p, node) &&
                   parkAndCheckInterrupt())
                   interrupted = true;
           }
       } finally {
           if (failed)
           //失败逻辑桶独占锁
               cancelAcquire(node);
       }
   }


```
2. 线程唤醒
```java
private void setHeadAndPropagate(Node node, int propagate) {
        //记录头结点以便检查
        Node h = head;
       //设置当前节点为新的头节点
        setHead(node);
        //节点满足如下条件则需要进行唤醒线程
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
           //如果节点s是空或者共享模式节点，那么就要唤醒共享锁上等待的线程
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

3. 具体的唤醒逻辑

```java
private void doReleaseShared() {
       for (;;) {
           Node h = head;
           //从头结点开始唤醒线程
           if (h != null && h != tail) {
               int ws = h.waitStatus;
               //后继节点需要被唤醒
               if (ws == Node.SIGNAL) {
                  // 将点h的状态设置成0，如果设置失败，就继续循环，再试一次
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                       continue;            // loop to recheck cases
                     //执行唤醒操作
                   unparkSuccessor(h);
               }
               //如果后续节点不需要被唤醒，则设置当前节点为PROPAGATE确保以后可以传递
               else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                   continue;                // loop on failed CAS
           }
           //头结点没有发生变化，则表示设置完成，退出循环
           if (h == head)                   // loop if head changed
               break;
       }
   }
```



```java
//唤醒node节点的后继节点
private void unparkSuccessor(Node node) {
      //节点的等待状态
       int ws = node.waitStatus;
       //置零当前线程所在的结点状态，允许失败
       if (ws < 0)
           compareAndSetWaitStatus(node, ws, 0);
      //后继节点
       Node s = node.next;
       //后继结点为空或者状态大于0，则无需处理
       if (s == null || s.waitStatus > 0) {
           s = null;
           //从尾节点遍历，找到下一个可唤醒的节点
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)
                   s = t;
       }
       if (s != null)
          //唤醒线程
           LockSupport.unpark(s.thread);
   }

```



```java
static final class Node {
        //节点在共享模式下的标记
        static final Node SHARED = new Node();
        //节点在独占锁模式下的标记
        static final Node EXCLUSIVE = null;
        //标志着线程被取消
        static final int CANCELLED =  1;
        //标志着后继线程(即队列链表的下一个节点)需要被阻塞.
        static final int SIGNAL    = -1;
        //标志着线程在Condition条件上等待阻塞
        static final int CONDITION = -2;
        //标志着下一个acquireShared方法线程应该被允许，即锁可向下一级传播。(用于共享锁)
        static final int PROPAGATE = -3;
        //维护锁的状态
        volatile int waitStatus;

        //前驱节点
        volatile Node prev;
        //后继结点      
        volatile Node next;
        //获取同步状态的线程,进入此节点队列的线程
        volatile Thread thread;
        //等待节点的后继节点。如果当前节点是共享的，那么这个字段是一个SHARED常量，也就是说节点类型（独占和共享）和等待队列中的后继节点共用一个字段。
        Node nextWaiter;

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        //返回前驱节点
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {   
        }
        //Used by addWaiter
        Node(Thread thread, Node mode) {
            this.nextWaiter = mode;
            this.thread = thread;
        }
        // Used by Condition
        Node(Thread thread, int waitStatus) {
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

###  独占锁
四维导图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711140810617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NjYxNzk4,size_16,color_FFFFFF,t_70)

1. 获取锁

```java
public final void acquire(int arg) {
     //尝试获取锁资源，失败则创建等待队列，此处的tryAcquire由子类进行实现
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        //中断当前线程
        selfInterrupt();
}
```

2. 将Node加入等待队列

```
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    //记录当前尾节点
    Node pred = tail;
    //此处为了提高性能，直接进行入队操作，失败则继续处理（多线程下竞争）
    if (pred != null) {
       //设置前驱节点
        node.prev = pred;
       // CAS操作，多个线程同时进行入队，只有一根线程能入队成功，其余的继续执行
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //上一步失败的节点入队
    enq(node);
    return node;
}
```

3. 节点加入队尾

```java
private Node enq(final Node node) {
//自旋锁，失败重试
    for (;;) {
    	//记录队尾元素
        Node t = tail;
        //初始化队列
        if (t == null) {
          //CAS操作，只有一个线程可以初始化头结点成功，其余的都要重复执行循环体
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
           //入队的节点指向队尾，此处会有多个节点指向队尾
            node.prev = t;
            //cas操作，保证多线程下只有一个节点真正指向队尾节点，其余失败重试
            if (compareAndSetTail(t, node)) {
                t.next = node;
                //该循环体唯一退出的操作，就是入队成功（否则就要无限重试）
                return t;
            }
        }
    }
}
```

4. 将节点加入等待队列后，先判断是否能够获取锁的资源，不能则进入挂起状态等待唤醒

```java
final boolean acquireQueued(final Node node, int arg) {
        //锁资源获取失败标记位
        boolean failed = true;
        try {
            //线程被中断标识
            boolean interrupted = false;
            // 自旋锁等待被唤醒
            for (;;) {
                //获取当前节点的前置节点
                final Node p = node.predecessor();
                //如果前置节点就是头结点，则尝试获取锁资源
                if (p == head && tryAcquire(arg)) {
                    //将当前节点设置为头结点
                    setHead(node);
                    p.next = null; //帮助GC
                    //表示锁资源成功获取
                    failed = false;
                    //返回中断标记，表示当前节点是被正常唤醒还是被中断唤醒
                    return interrupted;
                }
                //如果没有获取锁成功，则进入挂起逻辑
                //无论被中断或者正常唤醒，都会进行重新获取锁。成功则释放，失败则挂起
                // acquireInterruptibly不同在于，线程堵塞时被中断会抛出异常
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            //获取锁失败处理逻辑
            if (failed)
                cancelAcquire(node);
        }
    }
```



5. 检测节点状态，判断是否可进入挂起状态，同时剔除前边已经取消的节点

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取前置节点的waitStatus
        int ws = pred.waitStatus;
        //如果处于等待唤醒状态，则返回为true进行堵塞
        if (ws == Node.SIGNAL)
            return true;
        if (ws > 0) {
          //循环查找没有被取消的前节点，相当于把之前取消的节点从队列中剔除出去
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
       		//将前置节点状态设置Node.SIGNAL，执行上一步的自旋，进行堵塞
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```



6. 堵塞线程

```java
 private final boolean parkAndCheckInterrupt() {
      //堵塞线程
    LockSupport.park(this);
    //被唤醒之后，返回中断标记，即如果是正常唤醒则返回false，如果是由于中断醒来，就返回true
    return Thread.interrupted();
}
```



7. 获取锁或则加入等待队列失败后的处理

```java
private void cancelAcquire(Node node) {
        if (node == null)
            return;

        node.thread = null;

        Node pred = node.prev;
        // 跳过所有已经取消的前置节点，跟上面的那段跳转逻辑类似
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        //取得的未取消节点的后一节点
        Node predNext = pred.next;
        //把当前节点waitStatus置为取消，这样别的节点在处理时就会跳过该节点
        node.waitStatus = Node.CANCELLED;

        //如果当前是尾节点，则直接删除，即出队
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                //如果当前节点的前置节点不是头节点且它后面的节点等待它唤醒（waitStatus小于0），如果当前节点的后继节点没有被取消就把前置节点跟后置节点进行连接
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
```



8. 锁的释放

```java
public final boolean release(int arg) {
  // 调用tryRelease方法，尝试去释放锁，由子类具体实现
    if (tryRelease(arg)) {
        Node h = head;
        //如果队列头节点的状态不是0，那么队列中就可能存在需要唤醒的等待节点。
        //
        if (h != null && h.waitStatus != 0)
            // 唤醒线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

9. 线程唤醒

```java
private void unparkSuccessor(Node node) {

        int ws = node.waitStatus;
        //如果当前节点的状态小于0，则将该节点置为初始状态，标识节点已完成
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
              // 如果下一个节点为null，或者状态是已取消，那么就要寻找下一个非取消状态的节点
        if (s == null || s.waitStatus > 0) {
            s = null;
            // 从队列尾向前遍历，直到遍历到node节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
           //唤醒线程
            LockSupport.unpark(s.thread);
    }
```



 ### 共享锁的实现
 1.释放共享锁

```java
private void doAcquireShared(int arg) {
      // 创建一个共享锁节点
       final Node node = addWaiter(Node.SHARED);
       boolean failed = true;
       try {
           boolean interrupted = false;
           for (;;) {
              // 获取下一节点
               final Node p = node.predecessor();
               //前节点为头结点
               if (p == head) {
                 //尝试释放共享锁
                   int r = tryAcquireShared(arg);
                   if (r >= 0) {
                        //获取锁后的唤醒操作
                       setHeadAndPropagate(node, r);
                       p.next = null; // help GC
                       //如果是被中断唤醒，则设置中断标记为
                       if (interrupted)
                           selfInterrupt();
                       failed = false;
                       return;
                   }
               }
               //挂起逻辑同独占锁
               if (shouldParkAfterFailedAcquire(p, node) &&
                   parkAndCheckInterrupt())
                   interrupted = true;
           }
       } finally {
           if (failed)
           //失败逻辑桶独占锁
               cancelAcquire(node);
       }
   }

```

2. 线程唤醒

```java
private void setHeadAndPropagate(Node node, int propagate) {
        //记录头结点以便检查
        Node h = head;
       //设置当前节点为新的头节点
        setHead(node);
        //节点满足如下条件则需要进行唤醒线程
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
           //如果节点s是空或者共享模式节点，那么就要唤醒共享锁上等待的线程
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

3. 具体的唤醒逻辑

```java
private void doReleaseShared() {
       for (;;) {
           Node h = head;
           //从头结点开始唤醒线程
           if (h != null && h != tail) {
               int ws = h.waitStatus;
               //后继节点需要被唤醒
               if (ws == Node.SIGNAL) {
                  // 将点h的状态设置成0，如果设置失败，就继续循环，再试一次
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                       continue;            // loop to recheck cases
                     //执行唤醒操作
                   unparkSuccessor(h);
               }
               //如果后续节点不需要被唤醒，则设置当前节点为PROPAGATE确保以后可以传递
               else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                   continue;                // loop on failed CAS
           }
           //头结点没有发生变化，则表示设置完成，退出循环
           if (h == head)                   // loop if head changed
               break;
       }
   }
```



```java
//唤醒node节点的后继节点
private void unparkSuccessor(Node node) {
      //节点的等待状态
       int ws = node.waitStatus;
       //置零当前线程所在的结点状态，允许失败
       if (ws < 0)
           compareAndSetWaitStatus(node, ws, 0);
      //后继节点
       Node s = node.next;
       //后继结点为空或者状态大于0，则无需处理
       if (s == null || s.waitStatus > 0) {
           s = null;
           //从尾节点遍历，找到下一个可唤醒的节点
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)
                   s = t;
       }
       if (s != null)
          //唤醒线程
           LockSupport.unpark(s.thread);
   }
```


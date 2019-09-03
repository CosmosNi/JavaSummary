##  源码解析（线程等待）
1. await() 
```java
        public final void await() throws InterruptedException {
        // 线程被打断，抛出异常
            if (Thread.interrupted())
                throw new InterruptedException();
             //将节点加入等待序列
            Node node = addConditionWaiter();
            //释放当前线程所占用的lock，在释放的过程中会唤醒同步队列中的下一个节点
            // 此处很好理解，如果不释放所占用的锁，则由于condition会堵塞当前线程，导致，其他
            //线程永远获取不到锁，也就导致了死锁。
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            //自旋，检测节点是否已经进入堵塞队列
            while (!isOnSyncQueue(node)) {
               //线程挂起
                LockSupport.park(this);
                //线程被打断时退出自旋
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            // 被唤醒后，将进入阻塞队列，等待获取锁,即重入锁的lock
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
              //后续没有等待的下一节点，则清除该节点
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
            //线程中断的处理
                reportInterruptAfterWait(interruptMode);
        }

```

2. addConditionWaiter()
```java
		//加入等待队列
        private Node addConditionWaiter() {
        	//condition队尾元素
            Node t = lastWaiter;
            //如果队尾元素已经退出，则清除队列
            if (t != null && t.waitStatus != Node.CONDITION) {
            	//清除队列
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            //创建condition节点
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            //如果队尾尾为空，则说明是空队列
            if (t == null)
            	//将节点置为头结点
                firstWaiter = node;
            else
                t.nextWaiter = node;
             // 链接成功，将尾节点置为该节点
            lastWaiter = node;
            return node;
        }
```

3. unlinkCancelledWaiters()
```java
	//将已取消的所有节点清除出队列
        private void unlinkCancelledWaiters() {
            Node t = firstWaiter;
            Node trail = null;
            // 从头结点开始遍历，
            while (t != null) {
                Node next = t.nextWaiter;
                // 如果节点的状态不是CONDITION ，则说明该节点已取消
                if (t.waitStatus != Node.CONDITION) {
                	//断开连接
                    t.nextWaiter = null;
                    if (trail == null)
                    	//将下一个节点置为队首
                        firstWaiter = next;
                    else
                        trail.nextWaiter = next;
                    if (next == null)
                        lastWaiter = trail;
                }
                else
                    trail = t;
                t = next;
            }
        }
```

4. fullyRelease()
```java
final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            //获取锁的状态
            int savedState = getState();
            //释放锁，即重入锁的释放，锁释放失败则抛出异常
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
          //释放失败
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
```

5. isOnSyncQueue（）

```java
	//节点是否已经转移到阻塞队列
    final boolean isOnSyncQueue(Node node) {
    	// 如果当前节点的状态是CONDITION 或者没有前置节点，说明还没进入堵塞队列
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
         //如果有下一节点说明已经进入堵塞队列
        if (node.next != null) // If has successor, it must be on queue
            return true;
            //从尾部节点查找该节点
        return findNodeFromTail(node);
    }
```


##  源码解析（线程唤醒）
### 唤醒之signal() 
```java
        public final void signal() {
      	 // 调用 signal 方法的线程必须持有当前的独占锁
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
             // 找到头结点
            Node first = firstWaiter;
            if (first != null)
            	//唤醒头结点
                doSignal(first);
        }
```

2. ```java
    private void doSignal(Node first) {
                do {
                	//自旋，从头结点开始寻找第一个能被唤醒的节点
                    if ( (firstWaiter = first.nextWaiter) == null)
                        lastWaiter = null;
                    first.nextWaiter = null;
                } while (!transferForSignal(first) &&
                         (first = firstWaiter) != null);
            }`
    ```
    
    


### 唤醒之signalAll() 
```java
    public final void signalAll() {
    //同样检测
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignalAll(first);
    }

```

2. doSignalAll()

```java
        private void doSignalAll(Node first) {
            lastWaiter = firstWaiter = null;
            do {
            //自旋，从头到尾，依次唤醒
                Node next = first.nextWaiter;
                first.nextWaiter = null;
                transferForSignal(first);
                first = next;
            } while (first != null);
        }

```
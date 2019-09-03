# JUC之重入锁ReentrantLock

##   源码解析
### lock()
ReentrantLock是在AQS的基础上实现的独占锁。
1.  尝试加锁

```java
        final void lock() {
        	// 检测锁的状态，如果满足条件，则将当前锁的拥有者设置为为当前线程
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
            // 此处逻辑可参考上一篇AQS的文章
                acquire(1);
        }

```

2. 非公平锁方式下尝试获取锁

```java
        final boolean nonfairTryAcquire(int acquires) {
            //获取当前线程
            final Thread current = Thread.currentThread();
            // 获取当前锁的状态
            int c = getState();
            //如果状态为0，说明当前未有线程占有锁
            if (c == 0) {
            	// CAS操作，
                if (compareAndSetState(0, acquires)) {
                  // 设置锁的拥有着为当前线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //重入的逻辑
            // 同一线程再次获取锁
            else if (current == getExclusiveOwnerThread()) {
            	// 将锁的状态增加1
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

3. 锁的释放

```java
    public final boolean release(int arg) {
    	// 尝试释放锁
        if (tryRelease(arg)) {
            Node h = head;
            // 当头节点不为null以及该节点未被取消的情况下，唤醒线程
            if (h != null && h.waitStatus != 0)
            	//线程唤醒
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```
4. 具体的释放逻辑

```java
        protected final boolean tryRelease(int releases) {
        	//将当前锁的状态减1
            int c = getState() - releases;
            //如果占有锁的当前线程不是该线程则抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 锁的当前状态为0，即锁释放完成（考虑同一根线程加锁两次，需要释放两次）
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            //设置锁的状态
            setState(c);
            return free;
        }
```

5. 公平锁与非公平锁的区别

```java
       public final boolean hasQueuedPredecessors() {
        Node t = tail; 
        Node h = head;
        Node s;
        // 通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

### lockInterruptibly()
与lock的区别在与如下方法

```java
 private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                //  此处，lock只是返回中断状态，而lockInterruptibly则抛出异常
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
###  tryLock
与lock的区别在于在未获得锁的情况下，并未进行入等待队列的操作，而是直接返回结果。
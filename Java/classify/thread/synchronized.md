# synchronized和lock的区别

> 很多面试官爱问synchronized和lock的区别

面试官：区别？哈哈哈

我：面有表情，好的。

- **两者都是可重入锁**:两者都是可重入锁。“可重入锁”概念是：**自己可以再次获取自己的内部锁**。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。
- **synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API**:synchronized 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。ReenTrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。
- **ReenTrantLock 比 synchronized 增加了一些高级功能**
  1. **等待可中断**：过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
  2. **可实现公平锁**
  3. **可实现选择性通知（锁可以绑定多个条件）**：线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用notify/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知”
  4. **性能已不是选择标准**：在jdk1.6之前synchronized 关键字吞吐量随线程数的增加，下降得非常严重。1.6之后，**synchronized 和 ReenTrantLock 的性能基本是持平了。**

## synchronized修饰范围

面试官：先扯synchronized吧，修饰范围？可否了解？

我：当然了解。

- 实例方法：作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁
- 静态方法：作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
- 修饰代码块：指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁

这里可能让谈对象头里面都有什么哦，简单的说：**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的自身运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

## 底层原理

面试官：聊聊底层原理

我：好的，代码块和方法有些区别。

- 代码块

**synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。**

- 方法

**synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。**

面试官：1.6对synchronized的一些优化都有哪些？

我：JDK1.6对锁的实现引入了大量的优化，如**自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等**技术来减少锁操作的开销。

面试官：简单的说一下偏向锁、轻量级锁等。

- 偏向锁

引入偏向锁的目的和引入轻量级锁的目的很像，**他们都是为了没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗**。但是不同是：轻量级锁在无竞争的情况下**使用 CAS 操作去代替使用互斥量**。而偏向锁在**无竞争的情况下会把整个同步都消除掉**。

- 轻量级锁

轻量级锁不是为了代替重量级锁，**它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗，因为使用轻量级锁时，不需要申请互斥量**。另外，轻量级锁的加锁和解锁都用到了**CAS**操作。

- 锁消除

锁消除理解起来很简单，它指的就是虚拟机即使编译器在运行时，**如果检测到那些共享数据不可能存在竞争，那么就执行锁消除**。锁消除可以节省毫无意义的请求锁的时间。

- 自旋锁和自适应锁

**一般线程持有锁的时间都不是太长，所以仅仅为了这一点时间去挂起线程/恢复线程是得不偿失的。** 所以，虚拟机的开发团队就这样去考虑：“我们能不能让后面来的请求获取锁的线程等待一会而不被挂起呢？看看持有锁的线程是否很快就会释放锁”。**为了让一个线程等待，我们只需要让线程执行一个忙循环（自旋），这项技术就叫做自旋**。

**在 JDK1.6 中引入了自适应的自旋锁。自适应的自旋锁带来的改进就是：自旋的时间不在固定了，而是和前一次同一个锁上的自旋时间以及锁的拥有者的状态来决定**。

面试官：简要说一下偏向锁->轻量级锁->重量级锁的升级过程

我：好的

- 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁
- 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1
- 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。
- 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁
- 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
- 如果自旋成功则依然处于轻量级状态。
- 如果自旋失败，则升级为重量级锁。



> 个人觉得够了...能一口气扯完这些，足以了。AQS就够受了。
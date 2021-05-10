# java_读书05殷建卫架构笔记之java
并发编程基础:https://www.yuque.com/yinjianwei/vyrvkf/hfsk8i#14cba095



## sleep 时的线程状态

进入 TIMED_WAITING 状态的另一种常见情形是调用的 sleep 方法，单独的线程也可以调用，不一定非要有协作关系，当然，依旧可以将它视作为一种特殊的 wait/notify 情形。

这种情况下就是完全靠“自带闹钟”来通知了。

另：sleep(0) 跟 wait(0) 是不一样的，sleep 不存在无限等待的情况，sleep(0) 相当于几乎不等待。

需要注意，sleep 方法没有任何同步语义。通常，我们会说，sleep 方法不会释放锁。这种说法不能说说错了，但不是很恰当。

打个不太确切的比方，就好比你指着一个大老爷们说：“他下个月不会来大姨妈”，那么，我们能说你说错了吗？但是，显得很怪异。

就锁这个问题而言，确切的讲法是 sleep 是跟锁无关的。



## Daemon 线程

Daemon 线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这意味着，当一个 Java 虚拟机中不存在非 Daemon 线程的时候，Java 虚拟机将会退出。

注意：Daemon 属性需要在启动线程之前设置，不能在启动线程之后设置。

注意：在构建 Daemon 线程时，不能依靠 finally 块中的内容来确保执行关闭或清理资源的逻辑





## Thread类中interrupt
Thread类中interrupt（）、interrupted（）和isInterrupted（）方法详解:https://blog.csdn.net/qq_39682377/article/details/81449451

最后总结，关于这三个方法，interrupt（）是给线程设置中断标志；interrupted（）是检测中断并清除中断状态；isInterrupted（）只检测中断。还有重要的一点就是interrupted（）作用于当前线程，interrupt（）和isInterrupted（）作用于此线程，即代码中调用此方法的实例所代表的线程。

Java线程的传说(1)——中断线程Interrupted的用处：https://blog.csdn.net/budapest/article/details/6941802；"没有任何语言方面的需求要求一个被中断的程序应该终止。中断一个线程只是为了引起该线程的注意，被中断线程可以决定如何应对中断 "。


## Java线程等待唤醒机制
Java线程等待唤醒机制（加深理解）：https://blog.csdn.net/jdsjlzx/article/details/98470930

几个方法的比较

Thread.sleep(long millis)，一定是当前线程调用此方法，当前线程进入TIMED_WAITING状态，但不释放对象锁，millis后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。

Thread.yield()，一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield()不会导致阻塞。该方法与sleep()类似，只是不能由用户指定暂停多长时间。

thread.join()/thread.join(long millis)，当前线程里调用其它线程t的join方法，当前线程进入WAITING/TIMED_WAITING状态，当前线程不会释放已经持有的对象锁。线程t执行完毕或者millis时间到，当前线程一般情况下进入RUNNABLE状态，也有可能进入BLOCKED状态（因为join是基于wait实现的）。

obj.wait()，当前线程调用对象的wait()方法，当前线程释放对象锁，进入等待队列。依靠notify()/notifyAll()唤醒或者wait(long timeout) timeout时间到自动唤醒。

obj.notify()唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll()唤醒在此对象监视器上等待的所有线程。

LockSupport.park()/LockSupport.parkNanos(long nanos),LockSupport.parkUntil(long deadlines), 当前线程进入WAITING/TIMED_WAITING状态。对比wait方法,不需要获得锁就可以让线程进入WAITING/TIMED_WAITING状态，需要通过LockSupport.unpark(Thread thread)唤醒。

sleep()方法与yield()方法区别：

sleep()方法暂停当前线程后，会给其他线程执行机会，不会理会其他线程优先级。yield()方法只会给同优先级或更高优先级线程执行机会。

sleep()方法将当前线程转入阻塞状态，而yield()强制将当前线程转入就绪状态，因此完全可能某个线程调用yield()后立即再次获得CPU资源。

sleep()方法申明抛出InterruptException异常，要么捕捉要么显示抛出，而yield()没有申明抛出任何异常。

sleep()比yield()有更好的移植性，不建议yield()控制并发线程执行。


## 线程的安全性问题
https://www.yuque.com/yinjianwei/vyrvkf/ygfq5z

可见性、原子性、有序性


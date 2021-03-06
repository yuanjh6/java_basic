# java_并发编程07锁相关小结
锁相关小结

## 4种Java线程锁(线程同步)
### 1.synchronized
在Java中synchronized关键字被常用于维护数据一致性。
synchronized机制是给共享资源上锁，只有拿到锁的线程才可以访问共享资源，这样就可以强制使得对共享资源的访问都是顺序的。
Java开发人员都认识synchronized，使用它来实现多线程的同步操作是非常简单的，只要在需要同步的对方的方法、类或代码块中加入该关键字，它能够保证在同一个时刻最多只有一个线程执行同一个对象的同步代码，可保证修饰的代码在执行过程中不会被其他线程干扰。使用synchronized修饰的代码具有原子性和可见性，在需要进程同步的程序中使用的频率非常高，可以满足一般的进程同步要求。

```
synchronized (obj) {
//方法
…….
}
```
synchronized实现的机理依赖于软件层面上的JVM，因此其性能会随着Java版本的不断升级而提高。

到了Java1.6，synchronized进行了很多的优化，有适应自旋、锁消除、锁粗化、轻量级锁及偏向锁等，效率有了本质上的提高。在之后推出的Java1.7与1.8中，均对该关键字的实现机理做了优化。

需要说明的是，当线程通过synchronized等待锁时是不能被Thread.interrupt()中断的，因此程序设计时必须检查确保合理，否则可能会造成线程死锁的尴尬境地。

最后，尽管Java实现的锁机制有很多种，并且有些锁机制性能也比synchronized高，但还是强烈推荐在多线程应用程序中使用该关键字，因为实现方便，后续工作由JVM来完成，可靠性高。只有在确定锁机制是当前多线程程序的性能瓶颈时，才考虑使用其他机制，如ReentrantLock等。

### 2.ReentrantLock
可重入锁，顾名思义，这个锁可以被线程多次重复进入进行获取操作。

ReentantLock继承接口Lock并实现了接口中定义的方法，除了能完成synchronized所能完成的所有工作外，还提供了诸如可响应中断锁、可轮询锁请求、定时锁等避免多线程死锁的方法。

Lock实现的机理依赖于特殊的CPU指定，可以认为不受JVM的约束，并可以通过其他语言平台来完成底层的实现。在并发量较小的多线程应用程序中，ReentrantLock与synchronized性能相差无几，但在高并发量的条件下，synchronized性能会迅速下降几十倍，而ReentrantLock的性能却能依然维持一个水准。
因此我们建议在高并发量情况下使用ReentrantLock。

ReentrantLock引入两个概念：公平锁与非公平锁。

公平锁指的是锁的分配机制是公平的，通常先对锁提出获取请求的线程会先被分配到锁。反之，JVM按随机、就近原则分配锁的机制则称为不公平锁。

ReentrantLock在构造函数中提供了是否公平锁的初始化方式，默认为非公平锁。这是因为，非公平锁实际执行的效率要远远超出公平锁，除非程序有特殊需要，否则最常用非公平锁的分配机制。

ReentrantLock通过方法lock()与unlock()来进行加锁与解锁操作，与synchronized会被JVM自动解锁机制不同，ReentrantLock加锁后需要手动进行解锁。为了避免程序出现异常而无法正常解锁的情况，使用ReentrantLock必须在finally控制块中进行解锁操作。通常使用方式如下所示：

```
Lock lock = new ReentrantLock();
try {
lock.lock();
//…进行任务操作5 }
finally {
lock.unlock();
}
```
### 3.Semaphore
上述两种锁机制类型都是“互斥锁”，学过操作系统的都知道，互斥是进程同步关系的一种特殊情况，相当于只存在一个临界资源，因此同时最多只能给一个线程提供服务。但是，在实际复杂的多线程应用程序中，可能存在多个临界资源，这时候我们可以借助Semaphore信号量来完成多个临界资源的访问。
Semaphore基本能完成ReentrantLock的所有工作，使用方法也与之类似，通过acquire()与release()方法来获得和释放临界资源。

经实测，Semaphone.acquire()方法默认为可响应中断锁，与ReentrantLock.lockInterruptibly()作用效果一致，也就是说在等待临界资源的过程中可以被Thread.interrupt()方法中断。

此外，Semaphore也实现了可轮询的锁请求与定时锁的功能，除了方法名tryAcquire与tryLock不同，其使用方法与ReentrantLock几乎一致。Semaphore也提供了公平与非公平锁的机制，也可在构造函数中进行设定。

Semaphore的锁释放操作也由手动进行，因此与ReentrantLock一样，为避免线程因抛出异常而无法正常释放锁的情况发生，释放锁的操作也必须在finally代码块中完成。

### 4.AtomicInteger
首先说明，此处AtomicInteger是一系列相同类的代表之一，常见的还有AtomicLong、AtomicLong等，他们的实现原理相同，区别在与运算对象类型的不同。
我们知道，在多线程程序中，诸如++i

或

i++等运算不具有原子性，是不安全的线程操作之一。通常我们会使用synchronized将该操作变成一个原子操作，但JVM为此类操作特意提供了一些同步类，使得使用更方便，且使程序运行效率变得更高。通过相关资料显示，通常AtomicInteger的性能是ReentantLock的好几倍。

### 5. Java线程锁总结
1.synchronized：

在资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好。

2.ReentrantLock:

在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍，而ReentrantLock确还能维持常态。

高并发量情况下使用ReentrantLock。

3.Atomic:

和上面的类似，不激烈情况下，性能比synchronized略逊，而激烈的时候，也能维持常态。激烈的时候，Atomic的性能会优于ReentrantLock一倍左右。但是其有一个缺点，就是只能同步一个值，一段代码中只能出现一个Atomic的变量，多于一个同步无效。因为他不能在多个Atomic之间同步。

所以，我们写同步的时候，优先考虑synchronized，如果有特殊需要，再进一步优化。ReentrantLock和Atomic如果用的不好，不仅不能提高性能，还可能带来灾难。

以上就是Java线程锁的详解，除了从编程的角度应对高并发，更多还需要从架构设计的层面来应对高并发场景，例如：Redis缓存、CDN、异步消息等。


## java多线程互斥，和java多线程引入偏向锁和轻量级锁的原因？
**synchronized的重量级别的锁**，会频繁出现程序运行状态的切换，**线程的挂起和唤醒**，这样就会大量消耗资源，程序运行的效率低下。为了提高效率，jvm的开发人员，引入了偏向锁，和轻量级锁，尽量让多线程访问公共资源的时候，不进行程序运行状态的切换。

synchronized在jvm里实现原理,jvm基于进入和退出**Monitor**对象来实现方法同步和代码块同步的。在代码同步的开始位置织入monitorenter,在结束同步的位置（正常结束和异常结束处）织入monitorexit指令实现。线程执行到monitorenter处，将会获取锁对象对应的monitor的所有权，即尝试获得对象的锁。（任意对象都有一个monitor与之关联，当且一个monitor被持有后，他处于锁定状态）

java的**多线程安全是基于lock机制**实现的，而lock的性能往往不如人意。原因是，monitorenter与monitorexit这两个控制多线程同步的bytecode原语，是jvm依赖操作系统互斥（mutex）来实现的。

**互斥是一种会导致线程挂起**，并在较短时间内又需要重新调度回原线程的，较为消耗资源的操作。

为了优化java的Lock机制，从java6开始引入轻量级锁的概念。**轻量级锁本意是为了减少多线程进入互斥的几率**，并不是要替代互斥。它利用了cpu原语Compare-And-Swap（cas,汇编指令CMPXCHG）,尝试**进入互斥前，进行补救**。



### 为什么要自旋或者自适应自旋？
前面我们讨论互斥同步的时候，提到了**互斥同步对性能最大的影响是阻塞**的实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性能带来了很大的压力

共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得。

为了让线程等待，我们只须让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。

**自旋锁在JDK 1.4.2中就已经引入，只不过默认是关闭的**，

在JDK 1.6中就已经改为默认开启了。**自旋等待不能代替阻塞**，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的

**自旋等待的时间必须要有一定的限度**，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了。自旋次数的默认值是10次，用户可以使用参数-XX:PreBlockSpin来更改。

--->在JDK 1.6中引入了**自适应的自旋锁。自适应意味着自旋的时间不再固定**了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

有了自适应自旋，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越准确，虚拟机就会变得越来越“聪明”了。



### 锁削除
锁削除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到**不可能存在共享数据竞争的锁**进行削除。锁削除的主要判定依据来源于**逃逸分析的数据支持**（第11章已经讲解过逃逸分析技术），

也许读者会有疑问，变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是程序员自己应该是很清楚的，怎么会在明知道不存在数据争用的情况下要求同步呢？答案是有**许多同步措施并不是程序员自己加入的**，同步的代码在Java程序中的普遍程度也许超过了大部分读者的想象。比如：（只是说明概念，但实际情况并不一定如例子）在线程安全的环境中使用stringBuffer进行字符串拼加。则会在java文件编译的时候，进行锁销除。



### 锁粗化
原则上，将**同步块的作用范围限制得尽量小**——只在共享数据的实际作用域中才进行同步,如果存在锁竞争，那等待锁的线程也能尽快地拿到锁。

大部分情况下，上面的原则都是正确的，但是如果一系列的连续操作都**对同一个对象反复加锁和解锁**，甚至加锁操作是出现在循环体中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。

如果虚拟机探测到有这样**一串零碎的操作都对同一个对象加锁**，将会把加锁同步的范围扩展（锁粗化）到整个操作序列的外部。



### 锁的状态
锁一共有**四种状态（由低到高的次序）**：无锁状态，偏向锁状态，轻量级锁状态，重量级锁状态

锁的等级**只可以升级，不可以降级**。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率。

todo偏向锁升级降级流程（较复杂，先不管）


##  volatile和synchronized区别
volatile主要应用在多个线程对实例变量更改的场合，刷新主内存共享变量的值从而使得各个线程可以获得最新的值，线程读取变量的值需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。另外，synchronized还会创建一个内存屏障，内存屏障指令保证了所有CPU操作结果都会直接刷到主存中（即释放锁前），从而保证了操作的内存可见性，同时也使得先获得这个锁的线程的所有操作

volatile仅能使用在**变量级别**；synchronized则可以使用在变量、方法、和类级别的。

volatile**不会造成线程的阻塞**；synchronized可能会造成线程的阻塞，比如多个线程争抢synchronized锁对象时，会出现阻塞。

volatile仅能实现**变量的修改可见性**，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性，因为线程获得锁才能进入临界区，从而保证临界区中的所有语句全部得到执行。

volatile标记的变量**不会被编译器优化**，可以禁止进行指令重排；synchronized标记的变量可以被编译器优化。


synchronized不能防止指令重排,只有volatile变量是能实现禁止指令重排的

synchronized 虽然不能禁止指令重排，但也能保证有序性？

这个有序性是相对语义来看的，线程与线程间，每一个 synchronized 块可以看成是一个原子操作，它保证每个时刻只有一个线程执行同步代码，它可以解决上面引述的工作内存和主内存同步延迟现象引发的无序

所以，synchronized 和 volatile 的有序性与可见性是两个角度来看的：synchronized 是因为块与块之间看起来是原子操作，块与块之间有序可见

volatile 是在底层通过内存屏障防止指令重排的，变量前后之间的指令与指令之间有序可见

同时，synchronized 和 volatile 有序性不同也是因为其实现原理不同：synchronized 靠操作系统内核互斥锁实现的，相当于 JMM 中的 lock 和 unlock。退出代码块时一定会刷新变量回主内存

volatile 靠插入内存屏障指令防止其后面的指令跑到它前面去了

总而言之就是， synchronized 块里的非原子操作依旧可能发生指令重排


## synchronized和ReentrantLock有什么区别？
两者都是可重入锁；

synchronized依赖于JVM实现，ReentrantLock依赖于API实现；

synchronized不可以手动释放锁，ReentrantLock可以手动释放锁；

与synchronized相比，ReentrantLock增加了一些高级功能，比如线程的中断与等待、实现公平锁以及实现多个线程之间的通信；

ReentrantLock比synchronized更加灵活；


## ReentrantLock
ReentrantLock原理:https://blog.csdn.net/fuyuwei2015/article/details/83719444

一文彻底理解ReentrantLock可重入锁的使用:https://baijiahao.baidu.com/s?id=1648624077736116382&wfr=spider&for=pc





## 参考
synchronized使用和原理全解;https://blog.csdn.net/hebtu666/article/details/103057476#t2

什么是CAS机制，通俗易懂大白话版。：https://naurril.github.io/howtos/2018/08/22/inside_tfs.html

15.多线程编程中锁的4种状态-无锁状态 偏向锁状态 轻量级锁状态 重量级锁状态：https://blog.csdn.net/u014590757/article/details/79717549

synchronized和volatile区别：https://blog.csdn.net/xiaoming100001/article/details/79781680

Java synchronized 能防止指令重排序吗？：https://www.zhihu.com/question/337265532/answer/794398131

为什么volatile在并发下也是线程不安全的：https://blog.csdn.net/laifu007/article/details/89850299

能不能给我简单介绍一下 AtomicInteger 类的原理 ？：www.mianshigee.com/question/10035xxc

并发编程面试题之synchronized实现原理：https://blog.csdn.net/weixin_38251871/article/details/104667415


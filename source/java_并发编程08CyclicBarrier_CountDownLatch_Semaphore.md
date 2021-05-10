# java_并发编程08CyclicBarrier_CountDownLatch_Semaphore
JUC 中的同步器三个主要的成员：CountDownLatch、CyclicBarrier 和Semaphore。这三个是 JUC 中较为常用的同步器，通过它们可以方便地实现很多线程之间协作的功能。


## 一.闭锁CountDownLatch

### 场景
1）、运动会中赛跑项目，之后所有的赛跑运动员准备好了，此时裁判才能宣布该赛跑项目正式开始，裁判才能打出发信枪；当参与此次赛跑项目的所有的运动员都跑完了，此次赛跑项目才能算结束，才能统计出比赛名次。

。。。。等等

就是所有的准备好了，才能开始；或者是所有的都结束了，才能算结束。


### 基础
CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。

CountDownLatch类只提供了一个构造器：

```
public CountDownLatch(int count){  };//参数count为计数值
```
然后下面这3个方法是CountDownLatch类中最重要的方法：

```
public void await() thows InterruptedException{     };   //调用await()方法的线程会被挂起，它会等待直到count为0时才继续执行
public boolean await( long timeout,TimeUnit unit) throws InterruptedException{    };   //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown(){   };  //将count值-1
```
### 应用举例
CountDownLatch的用法

```
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
          
         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}

```
执行结果：

```
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程 
```



## 二.栅栏CyclicBarrier
### 场景
适合的业务场景，比如

1）、，现有一大任务，需要得到全年的统计数据的，这个工作量是巨大的，那么可以将其分割为12个月的子任务，各个子任务相互独立，当所有子任务完成了，则就可以进行全年统计了，这样大大提升了统计效率。

2）、大家一起去郊游，由于大家住的地方比较分散，故需要一个集合点之后一起出发，这样大家才能玩得开心嘛。

就是当有一个大任务时，需要分配多个子任务去执行，只有当所有的子任务都执行完成后，才能执行主任务。


### 基础
回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。　　
CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：　　
```
public CyclicBarrier(int parties, Runnable barrierAction){    }
public CyclicBarrier(int parties){     }
```
参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：

```
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```
第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。


### 应用举例
假若有若干个线程都要进行写数据操作，并且只有所有线程都完成写数据操作之后，这些线程才能继续做后面的事情，此时就可以利用CyclicBarrier了：

```
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);
        for(int i=0;i<N;i++)
            new Writer(barrier).start();
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
 
        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println("所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```
执行结果：

```
线程Thread-0正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1正在写入数据...
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-1写入数据完毕，等待其他线程写入完毕
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
所有线程写入完毕，继续处理其他任务...
```
从上面输出结果可以看出，每个写入线程执行完写数据操作之后，就在等待其他线程写入操作完毕。

当所有线程线程写入操作完毕之后，所有线程就继续进行后续的操作了。

如果说想在所有线程写入操作完之后，进行额外的其他操作可以为**CyclicBarrier提供Runnable参数**：

```
int N = 4;
CyclicBarrier barrier  = new CyclicBarrier(N,new Runnable() {
    @Override
    public void run() {
        System.out.println("当前线程"+Thread.currentThread().getName());   
    }
});
```
当四个线程都到达barrier状态后，会由最后一个线程去执行Runnable。

另外CyclicBarrier是可以重用的,而CountDownLatch无法进行重复使用。

## 三.信号量Semaphore
### 场景
适合的业务场景，比如
1）、当有10个人去上茅厕，但是只有5个坑，即只能同时5个人使用，只有当一个人不使用坑了，另一个人才能使用该空闲的坑，一直维持着只能同时5个人使用。

2）、当停车场来了100辆车时，但是只有30个停车位，即只能同时提供30个车辆停放，只有当一辆车开走了，另一辆车才能进入该空闲的停车位，一直维持着只能同时提供30个停车位。

就是同时只能提供有限的，走一个才能进一个。

### 基础
Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

```
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```
下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：

```
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```
acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```
另外还可以通过availablePermits()方法得到可用的许可数目。


## 四.总结
1）CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

2）Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。


## 参考

CyclicBarrier、CountDownLatch与Semaphore的小记:https://www.cnblogs.com/xiaoxian1369/p/5394733.html

CountDownLatch、CyclicBarrier、Semaphore的区别:https://blog.csdn.net/koobee1/article/details/79606816  
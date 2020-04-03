---
title: 同步与并发机制--Java实现
date: 2020-2-27 16:11:20
categories: 
- 多线程
tags: 
- java
- 并发
- 笔记
- leetcode
---

## 无竞争并发

* 加锁机制
* 确保关键部分代码对cpu的独占性
* 加锁后其他线程访问该资源时被阻塞

### synchronized机制

* 悲观锁；
* JVM会为一个使用内部锁（synchronized）的对象维护两个集合，`Entry Set`锁池和`Wait Set`等待池；
* 线程A占有该锁时，其他试图获得锁的线程进入`Entry Set`；
* 若线程A调用被加锁对象的wait()方法，线程A释放锁，并进入 `Wait Set`；
* 该锁被释放时，`Entry Set`中的一个线程获得该锁，进入 `Runable` 状态；
* 当A调用被加锁对象的notify()方法时，`Wait Set` 中的一个线程被唤醒，进入`Entry Set`，和此时在`Entry Set`中的其他线程竞争该锁；
* 当A调用被加锁对象的notifyall()方法时，`Wait Set` 中的所有线程被唤醒，进入`Entry Set`，和此时在`Entry Set`中的其他线程竞争该锁；
* 拥有锁的线程，Thread.sleep()不会释放锁
* synchronized可以作用于代码块和方法
  
下面是一些练习

**注意**在线程屏障类问题中，判断某线程是否需要wait()的条件应该是

```java
while(condition){
    lock.wait()
}
```

这里只能是`while`而不能是`if`,因为`if`只判断一次，当它再次得到锁之后就不再判断这个条件时是否满足，直接进行下去了。而我们需要的是只有这个条件满足，它才能获得锁，继续进行下面的操作，所以这里只能是`while`的。

**leetcode 1114** wait and notify机制实现。有三个线程分别调用同一个Foo对象的first(),sencond(),third()方法，要求三个方法中的代码块顺序执行输出firstsecondthird

```java
class Foo {
    Boolean finishFirst=false,finishSecond=false;
    Object lock =new Object();
    public Foo() {
    }
    public void first(Runnable printFirst) throws InterruptedException {
        synchronized(lock) {
            // printFirst.run() outputs "first".
            printFirst.run();
            finishFirst=true;
            lock.notifyAll();
        }
    }
    public void second(Runnable printSecond) throws InterruptedException {
        synchronized (lock) {
            while(finishFirst==false){
                lock.wait();
            }
            // printSecond.run() outputs "second".
            printSecond.run();
            finishSecond=true;
            lock.notifyAll();
        }
    }
    public void third(Runnable printThird) throws InterruptedException {
        synchronized (lock) {
            while(finishSecond==false){
                lock.wait();
            }
            // printThird.run() outputs "third".
            printThird.run();
        }
    }
}
```

**leetcode 1115**,要求交替输出foo和bar

```java
class FooBar {
    private int n;
    public FooBar(int n) {
        this.n = n;
    }
    int f=0;
    Object lock=new Object();
    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            // printFoo.run() outputs "foo". Do not change or remove this line.
            synchronized (lock) {
                while(f==1){
                    lock.wait();
                }
                printFoo.run();
                f=1;
                lock.notify();
            }
        }
    }
    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            synchronized (lock) {
                // printBar.run() outputs "bar". Do not change or remove this line.
                while(f==0){
                    lock.wait();
                }
                printBar.run();
                f=0;
                lock.notify();
            }
        }
    }
}
```

**leetcode 1116**,交替打印0102030405060708到n

```java
import java.util.function.IntConsumer;
class ZeroEvenOdd {
    private int n;
    public ZeroEvenOdd(int n) {
        this.n = n;
    }
    Object lock=new Object();
    // printNumber.accept(x) outputs "x", where x is an integer.
    int f=0;
    int e=1;
    public void zero(IntConsumer printNumber) throws InterruptedException {
        for(int j=0;j<n;j++){
            synchronized (lock) {
                while(f==1){
                    lock.wait();
                }
                printNumber.accept(0);
                f=1;
                lock.notifyAll();
            }
        }
    }
    public void even(IntConsumer printNumber) throws InterruptedException {
        for(int i=2;i<=n;i+=2){
            synchronized (lock){
                while(f==0||e==1){
                    lock.wait();
                }
                printNumber.accept(i);
                e=1;
                f=0;
                lock.notifyAll();
            }
        }
    }
    public void odd(IntConsumer printNumber) throws InterruptedException {
        for(int i=1;i<=n;i+=2) {
            synchronized (lock) {
                while(f==0||e==0){
                    lock.wait();
                }
                printNumber.accept(i);
                f=0;
                e=0;
                lock.notifyAll();
            }
        }
    }
}
```

**leetcode 1117**，交替打印两个H和一个O，如果一个氧线程到达屏障时没有氢线程到达，它必须等候直到两个氢线程到达。

如果一个氢线程到达屏障时没有其它线程到达，它必须等候直到一个氧线程和另一个氢线程到达

```java
class H2O {

    public H2O() {

    }
    Object lock=new Object();
    int H=0,O=0;
    public void hydrogen(Runnable releaseHydrogen) throws InterruptedException {
        synchronized (lock) {
            while(H==2){
                lock.wait();
            }
            // releaseHydrogen.run() outputs "H". Do not change or remove this line.
            releaseHydrogen.run();
            H++;
            if(H==2&&O==1){
                H=0;
                O=0;
            }
            lock.notifyAll();
        }
    }

    public void oxygen(Runnable releaseOxygen) throws InterruptedException {
        synchronized (lock) {
            while(O==1){
                lock.wait();
            }
            // releaseOxygen.run() outputs "O". Do not change or remove this line.
            releaseOxygen.run();
            O++;
            if(H==2&&O==1){
                H=0;
                O=0;
            }
            lock.notifyAll();
        }
    }
}
```

### 信号量机制（semaphore）

* 信号量是一种控制线程对公共资源的访问的变量，可用于解决临界区问题；
* 是一种无锁机制
* 这种机制的实现基于CPU提供的原子操作`CAS`（compare and swap)，让信号量的值一定是原子的递增或递减；

下面是一些练习。

**leetcode 1114** 信号量机制实现。

```java
import java.util.concurrent.Semaphore;

class Foo {

    Semaphore finishFirst = new Semaphore(0);
    Semaphore finishSecond = new Semaphore(0);

    public Foo() {
    }

    public void first(Runnable printFirst) throws InterruptedException {
        // printFirst.run() outputs "first".
        printFirst.run();
        finishFirst.release();
    }

    public void second(Runnable printSecond) throws InterruptedException {
        // printSecond.run() outputs "second".
        finishFirst.acquire();
        printSecond.run();
        finishSecond.release();
    }

    public void third(Runnable printThird) throws InterruptedException {
        // printThird.run() outputs "third".
        finishSecond.acquire();
        printThird.run();
    }
}
```

**leetcode 1226**，哲学家吃饭问题，建立一个保存每个位置的叉子数量的信号量数组，保存每个位置的叉子数，每次先拿起左边的叉子，若不能则让线程睡眠1ms然后重复这一过程，再拿起右边的叉子，若不能则放下左边的叉子，让线程睡1ms然后重复这一过程，直到可以同时拿起左右两只叉子，拿到后就吃面，然后跳出循环。这里在请求失败时若不使用Thread.sleep(1)会有两三个测试点超时，这是因为这个线程频繁的请求其他线程没有释放的资源而占用CPU，造成了资源浪费。

```java
import java.util.concurrent.Semaphore;

class DiningPhilosophers {
    static Semaphore[] forks=new Semaphore[5];
    public DiningPhilosophers() {
        for(int i=0;i<5;i++){
            forks[i]=new Semaphore(1);
        }
    }

    // call the run() method of any runnable to execute its code
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
        while (true) {
            int f = 0;
            if (forks[(philosopher + 4) % 5].tryAcquire()) {
                f = 1;
            }
            if (forks[philosopher].tryAcquire()) {
                if (f == 1) {
                    pickLeftFork.run();
                    pickRightFork.run();
                    eat.run();
                    putLeftFork.run();
                    forks[philosopher].release();
                    putRightFork.run();
                    forks[(philosopher + 4) % 5].release();
                    break;
                } else {
                    forks[philosopher].release();
                    Thread.sleep(1);
                }
            } else {
                if(f==1){
                    forks[(philosopher + 4) % 5].release();
                    Thread.sleep(1);
                }
            }
        }
    }
}
```

### volatile关键字

* 直接从内存中获取变量的值，而不是从从缓存中，可以保证内存可见性。
* 加锁会影响效率，要保证效率的情况下可以使用volatile关键字
* 当写一个volatile变量时,JVM会把该线程对应的本地内存中的共享变量刷新到主内存。
* 当读一个volatile变量时,JVM会把该线程对应的本地内存置为无效
* 是

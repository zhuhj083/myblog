---
title: 线程交替打印ABC的多种实现方法
date: 2019-03-14 17:18:18
tags: [java 并发 线程]
categories: java
---

# 题目描述
建立三个线程A、B、C，A线程打印10次字母A，B线程打印10次字母B,C线程打印10次字母C，但是要求三个线程同时运行，并且实现交替打印，即按照ABCABCABC的顺序打印。


# 1、Synchronized同步法
```java
public class ThreadABC_Notify {

    static class MyThread extends Thread{

        private String name;
        private Object prev;
        private Object self;

        public MyThread(String name, Object prev, Object self) {
            this.name = name;
            this.prev = prev;
            this.self = self;
        }

        /**
         * wait() 与 notify/notifyAll() 是Object类的方法，在执行两个方法时，要先获得锁。
         * 当线程执行wait()时，会把当前的锁释放，然后让出CPU，进入等待状态。
         * 当执行notify/notifyAll方法时，会唤醒一个处于等待该 对象锁 的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域（synchronized修饰的代码块）后再释放锁。
         * 从这里可以看出，notify/notifyAll()执行后，并不立即释放锁，而是要等到执行完临界区中代码后，再释放。
         * 所以在实际编程中，我们应该尽量在线程调用notify/notifyAll()后，立即退出临界区。即不要在notify/notifyAll()后面再写一些耗时的代码。
         */
        @Override
        public void run() {
            int count = 10 ;
            while (count > 0 ){
                synchronized (prev) {
                    synchronized (self){
                        System.out.println(name);
                        count-- ;
                        self.notifyAll();   // 唤醒其他线程竞争self锁，注意此时self锁并未立即释放。
                    }
                    try {
                        if (count == 0){    // 如果count==0,表示这是最后一次打印操作，通过notifyAll操作释放对象锁。
                            prev.notifyAll();
                        }else{              // 立即释放 prev锁，当前线程休眠，等待唤醒
                            prev.wait();
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();

        Thread A = new MyThread("A",c,a);
        Thread B = new MyThread("B",a,b);
        Thread C = new MyThread("C",b,c);

        A.start();
        Thread.sleep(10);
        B.start();
        Thread.sleep(10);
        C.start();
    }

}

```

# 2、Lock Condition 法

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * private Lock lock = new ReentrantLock();
 * private Condition condition = lock.newCondition();
 * condition.await();//this.wait();
 * condition.signal();//this.notify();
 * condition.signalAll();//this.notifyAll();
 *
 */

public class ThreadABC_Condition {

    private static Lock lock = new ReentrantLock();

    private static Condition A = lock.newCondition();
    private static Condition B = lock.newCondition();
    private static Condition C = lock.newCondition();

    private static int count = 0 ;

    static class ThreadA extends Thread {
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 0 ){
                        A.await();
                    }
                    System.out.print("A");
                    count++;
                    B.signal(); // A执行完唤醒B线程
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    static class ThreadB extends Thread {
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 1 ){
                        B.await();
                    }
                    System.out.print("B");
                    count++;
                    C.signal();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    static class ThreadC extends Thread {
        @Override
        public void run() {
            try {
                lock.lock();
                for (int i = 0; i < 10; i++) {
                    while (count % 3 != 2 ){
                        C.await();
                    }
                    System.out.print("C");
                    count++;
                    A.signal();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ThreadB().start();
        new ThreadA().start();
        new ThreadC().start();
    }
}



```

# 3、Semaphore法
```java
import java.util.concurrent.Semaphore;

/**
 * Semaphore又称信号量，是操作系统中的一个概念，在Java并发编程中，信号量控制的是线程并发的数量。
 *
 * public Semaphore(int permits);
 * 其中参数permits就是允许同时运行的线程数目;
 *
 * Semaphore semaphore = new Semaphore(10,true);
 * semaphore.acquire();
 * //do something here
 * semaphore.release();
 */
public class ThreadABC_Semaphore {

    private static Semaphore A = new Semaphore(1);
    private static Semaphore B = new Semaphore(0);
    private static Semaphore C = new Semaphore(0);

    static class ThreadA extends Thread{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    A.acquire();
                    System.out.print("A");
                    B.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class ThreadB extends Thread{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    B.acquire();
                    System.out.print("B");
                    C.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    static class ThreadC extends Thread{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    C.acquire();
                    System.out.print("C");
                    A.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread a = new ThreadA();
        Thread b = new ThreadB();
        Thread c = new ThreadC();
        a.start();
        c.start();
        b.start();
    }

}

```
